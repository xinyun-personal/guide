## Ubuntu Server Setup

## First need to update ubuntu

```bash

sudo apt update
```

## Nginx

```bash

sudo apt install nginx -y;

```

```bash

sudo ufw app list;
sudo ufw allow 'Nginx HTTP';
sudo ufw enable;
```

> Setting Nginx Port
>> Using reverse proxy only need to use `Nginx HTTP`
> Default `Nginx HTTPS`

## Phpmyadmin

```bash

sudo apt update;
sudo apt install phpmyadmin -y;
sudo apt install php8.*-fpm php-mysql;
```

> If install phpmyadmin got error
>> `sudo mysql;` Or `sudo mysql -u root -p;`

> Remove Password Validate Component
>> `UNINSTALL COMPONENT "file://component_validate_password"`

> Exit MYSQL
>> `Exit`

> Run install phpmyadmin command

## Nodejs & NPM

```bash

sudo apt update;
sudo apt install nodejs npm -y;
sudo npm cache clean -f;
sudo npm install -g n;
sudo n stable;
```

## PM2

```bash

sudo apt update;
sudo npm install pm2 -g;
sudo pm2 completion install;
```

## MYSQL-Server

> Setting
>> VALIDATE PASSWORD COMPONENT `Y`

> > VALIDATE PASSWORD POLICY `2`

> > DISALLOW Root Remote Login `Y`

> > RELOAD Privilege `Y`

```bash

sudo apt install mysql-server -y;
sudo mysql_secure_installation;
```

## Default Vuejs static file nginx config

> Vuejs or Static File

```text
server {
    listen 80;
    charset utf-8;
    client_max_body_size 50M;
    server_name your-server-ip-or-domain;

    root /var/www/html/vue-app;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }
    
    error_log  /var/log/nginx/vue-app-error.log;
    access_log /var/log/nginx/vue-app-access.log;
}
```

## Default Reverse Proxy

```text
server {
    client_max_body_size 50M;
    server_name  your-server-ip-or-domain;

    location / {
            proxy_pass http://127.0.0.1:2001;
            include /etc/nginx/proxy_params;
            proxy_buffers 8 1024k;
            proxy_buffer_size 1024k;
    }
}

```
