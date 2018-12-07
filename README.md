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
- IP: AWS public IP

## Working Directory
`cd /var/www/project_folder`
e.g. `cd /var/www/labelingflask`

### with virtualenv
- create virtual environment `virtualenv -p /usr/bin/python3 venv`
- activate `source venv/bin/activate`
- deactivate `deactivate`

## Installation
### python3
```python
  sudo apt-get update
  sudo apt-get install python3-pip python3-dev nginx
```
### Flask & uwsgi
```python
  pip install uwsgi flask
```

## Files Arrangement
### Flask
- put your .py(e.g. labelingServer.py) at `/var/www/labelingflask`

### uwsgi
#### Test uwsgi
`cd /var/www/labelingflask`
- `$ uwsgi --module labelingServer --callable app --http :1234 --venv /var/www/labelingflask/venv` </br>
you can test at http://IP:1234 to see whether it works well.

### nginx
- `sudo vim /etc/nginx/sites-enabled/labelingflask.conf`</br>
listen on port 8080, you can choose your own port.</br>
`uwsgi_pass` is the same as socket in uwsgi, we will use it later.
```
  server {
    listen 8080;
    server_name server_domain_or_IP;
    root  /var/www/labelingflask;
        location / {
                try_files $uri @uwsgi;
        }
        location @uwsgi{
                include uwsgi_params;
                uwsgi_pass 127.0.0.1:6000;
        }
  }
```
- `sudo service nginx start` to start nginx
- `sudo service nginx status` to see the status of nginx
- `cat /var/log/nginx/error.log` to see error log
- `cat /var/log/nginx/access.log` to see successiful log

### Test connection between uwsgi & nginx
- `$ uwsgi --module labelingServer --callable app --socket 127.0.0.1:6000 --chown-socket www-data:www-data --venv /var/www/labelingflask/venv` </br>
you can test at http://IP:8080 to see whether it works well.

#### Errors with uwsgi
- error: `!!! no internal routing support, rebuild with pcre support !!!` </br>
solution: `pip install uwsgi -I --no-cache-dir`

### Command to File: uwsgi.ini
- create `uwsgi.ini` at `/etc/uwsgi/apps-enabled/uwsgi.ini`
```
  [uwsgi]
  module = labelingServer
  callable = app
  socket = 127.0.0.1:6000
  chdir = /var/www/labelingflask
  venv = /var/www/labelingflask/venv
  logto = /var/www/labelingflask/uwsgi.log
  processes = 2
  master = true
  chown-socket = www-data:www-data
  chmod-socket = 660
  vacuum = true
```
- run `$ uwsgi --ini /etc/uwsgi/apps-enabled/uwsgi.ini` to see whether it works.

### set uwsgi to system service
If you run `uwsgi --ini /etc/uwsgi/apps-enabled/uwsgi.ini`, after closing the terminal, uwsgi will be closed too.
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
  WorkingDirectory=/var/www/labelingflask
  Environment=FLASKR_SETTINGS=/var/www/labelingflask/venv
  ExecStart=/var/www/labelingflask/venv/bin/uwsgi --ini /etc/uwsgi/apps-available/lablingflask.ini

  [Install]
  WantedBy=multi-user.target
```
- `sudo systemctl start labeling` to start uwsgi
- `sudo systemctl status labeling` to check the status
- `systemctl daemon-reload` to reload system service!!!

### TEST
After start uwsgi & nginx, go to `IP:8080` (your port) to see whether it works well.
