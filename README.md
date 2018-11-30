# flask-uwsgi-nginx

## Description

We want to use mongodb for backend collecting data from Android-app.
To make the server more stable, here we use flask+uwsgi+nginx.

- `Flask` python web framework, define how to handle each request and connect to mongodb.
- `uwsgi` define how to connect flask framework and web server.
- `nginx` web server, receives responds from flask and requests from client.

## Platform & Environment
- ubuntu 16.04.3 LTS
- python3.5

## Installation
path `/home/ubuntu`

### python3
```python
  sudo apt-get update
  sudo apt-get install python3-pip python3-dev nginx
```
### Flask & uwsgi
```python
  pip install uwsgi flask
```

## Files arrangement
### Flask
- put your .py(e.g. server.py) at `/home/ubuntu`

### uwsgi
- create `uwsgi.ini` at `/home/ubuntu`
```
  [uwsgi]
  module = server:app
  socket = 127.0.0.1:9000
  chdir = /home/ubuntu
  logto = /home/ubuntu/server.log
  processes = 2
  master = true
  chmod-socket = 660
  vacuum = true
```
- run `uwsgi --ini uwsgi.ini` to see whether it works normally

### nginx
- `sudo vim /etc/nginx/sites-available/default`</br>
listen on port 80 (default), if it is not idle, change to other port (e.g. 8080).</br>
`uwsgi_pass` is the same as socket in uwsgi.
```
  server {
    listen 80;
    server_name server_domain_or_IP;
    location / {
      include uwsgi_params;
      uwsgi_pass 127.0.0.1:9000
    }
  }
```
- `sudo service nginx start` to start nginx
- `sudo service nginx status` to see the status of nginx
- `cat /var/log/nginx/error.log` to see error log

### set uwsgi to system service
If you run `uwsgi --ini myproject.ini`, after closing the terminal, uwsgi will be closed either.
So we will use `systemd` to make uwsgi a system service.
- `sudo vim /etc/systemd/system/labeling.service` (for labeling Study)
be careful to set the right path for `ExecStart`
```
  [Unit]
  Description=uWSGI instance to serve labelingStudy
  After=network.target

  [Service]
  User=ubuntu
  Group=www-data
  WorkingDirectory=/home/ubuntu
  Environment=FLASKR_SETTINGS=/home/ubuntu
  ExecStart=/home/ubuntu/.local/bin/uwsgi --ini /home/ubuntu/uwsgi.ini

  [Install]
  WantedBy=multi-user.target
```
- `sudo systemctl start labeling` to start uwsgi
- `sudo systemctl status labeling` to check the status

### TEST
After start uwsgi & nginx, go to `IP:80` (your port) to see whether it works well.</br>
you can see log at `/home/ubuntu/server.log`.
