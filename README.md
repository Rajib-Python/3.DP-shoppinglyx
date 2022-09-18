# shoppinglyx
This is a Static DJango Shopping Website 



### Step 1 — Installing Packages from the Ubuntu Repositories


```

sudo apt update

sudo apt install python3-pip python3-dev libpq-dev postgresql postgresql-contrib nginx curl

or 

sudo apt install python-pip python-dev libpq-dev postgresql postgresql-contrib nginx curl

```


### Step 2 — Creating the PostgreSQL Database and User

```
sudo -u postgres psql
CREATE DATABASE shoppinglyx;

CREATE USER shoppinglyxuser WITH PASSWORD 'password';
ALTER ROLE shoppinglyxuser SET client_encoding TO 'utf8';
ALTER ROLE shoppinglyxuser SET default_transaction_isolation TO 'read committed';
ALTER ROLE shoppinglyxuser SET timezone TO 'UTC';

GRANT ALL PRIVILEGES ON DATABASE shoppinglyx TO shoppinglyxuser;
\q

```



```

DB_ENGINE=django.db.backends.postgresql

DB_NAME=shoppinglyx
DB_USER=shoppinglyxuser
DB_PASSWORD=password
DB_HOST=localhost
DB_PORT=5432

```



### Step 3 — Creating a Python Virtual Environment for your Project

```
sudo -H pip3 install --upgrade pip
sudo -H pip3 install virtualenv
sudo -H pip install --upgrade pip

```

```

virtualenv venv
source venv/bin/activate
pip install django gunicorn psycopg2-binary

```

```
cd shoppinglyx
python manage.py createsuperuser
python3 manage.py collectstatic

sudo ufw allow 8000
python manage.py runserver 0.0.0.0:8000

```

### Step 4 — Testing Gunicorn’s Ability to Serve the Project

```
cd ~/shoppinglyx
gunicorn --bind 0.0.0.0:8000 shoppinglyx.wsgi
deactivate

```

## Step 7 — Creating systemd Socket and Service Files for Gunicorn


### sudo vim /etc/systemd/system/gunicorn.socket

```
[Unit]
Description=gunicorn socket

[Socket]
ListenStream=/run/gunicorn.sock

[Install]
WantedBy=sockets.target
```

### sudo vim /etc/systemd/system/gunicorn.service

```

[Unit]
Description=gunicorn daemon
Requires=gunicorn.socket
After=network.target

[Service]
User=ubuntu
Group=www-data
WorkingDirectory=/home/ubuntu/shoppinglyx
ExecStart=/home/ubuntu/venv/bin/gunicorn \
          --access-logfile - \
          --workers 3 \
          --bind unix:/run/gunicorn.sock \
          shoppinglyx.wsgi:application

[Install]
WantedBy=multi-user.target


```

```
sudo systemctl start gunicorn.socket
sudo systemctl enable gunicorn.socket
sudo systemctl status gunicorn.socket

```


```
sudo systemctl daemon-reload
file /run/gunicorn.sock
curl --unix-socket /run/gunicorn.sock localhost
sudo systemctl status gunicorn
sudo systemctl restart gunicorn


```



## Step 10 — Configure Nginx to Proxy Pass to Gunicorn


### sudo vim /etc/nginx/sites-available/shoppinglyx



```
server {
    listen 80;
    server_name 54.205.54.141;

    location = /favicon.ico { access_log off; log_not_found off; }
    location /static/ {
        root /home/ubuntu/shoppinglyx;
    }

    location / {
        include proxy_params;
        proxy_pass http://unix:/run/gunicorn.sock;
    }
}


```



### sudo ln -s /etc/nginx/sites-available/shoppinglyx /etc/nginx/sites-enabled

```

sudo nginx -t
sudo systemctl restart nginx
sudo ufw delete allow 8000
sudo ufw allow 'Nginx Full'

```



![alt text](https://github.com/geekyshow1/shoppinglyx/blob/main/Screenshots/Home.jpeg)
