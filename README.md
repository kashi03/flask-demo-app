# flask-demo-app

ubuntu 20.04  

```
git clone https://github.com/kashi03/flask-demo-app.git
cd flask-demo-app/
sudo pip install -r requirements.txt 
sudo apt install nginx gunicorn
sudo vim /etc/systemd/system/gunicorn.service
sudo vim /etc/systemd/system/gunicorn.socket
sudo vim /etc/nginx/conf.d/default.conf
sudo systemctl enable --now gunicorn.socket
sudo systemctl restart nginx
```

/etc/systemd/system/gunicorn.service

```
[Unit]
Description=gunicorn daemon
Requires=gunicorn.socket
After=network.target

[Service]
Type=notify
# the specific user that our service will run as
User=www-data
Group=www-data
# another option for an even more restricted service is
# DynamicUser=yes
# see http://0pointer.net/blog/dynamic-users-with-systemd.html
RuntimeDirectory=gunicorn
WorkingDirectory=/home/kashi/flask-demo-app
ExecStart=/usr/bin/gunicorn app:app
ExecReload=/bin/kill -s HUP $MAINPID
KillMode=mixed
TimeoutStopSec=5
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```
  
/etc/systemd/system/gunicorn.socket

```
[Unit]
Description=gunicorn socket

[Socket]
ListenStream=/run/gunicorn.sock
# Our service won't need permissions for the socket, since it
# inherits the file descriptor by socket activation
# only the nginx daemon will need access to the socket
SocketUser=www-data
# Optionally restrict the socket permissions even more.
# SocketMode=600

[Install]
WantedBy=sockets.target
```

/etc/nginx/conf.d/default.conf

```
server {
    listen 80;
    server_name 172.20.44.52;
    location / {
        proxy_pass http://unix:/run/gunicorn.sock;
    }
}
```