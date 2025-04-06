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

>> VALIDATE PASSWORD POLICY `2`

>> DISALLOW Root Remote Login `Y`

>> RELOAD Privilege `Y`

```bash

sudo apt install mysql-server -y;
sudo mysql_secure_installation;
```