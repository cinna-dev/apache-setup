# Apache-setup


## Installation

```bash
sudo apt-get install apache2
```

or 

```bash
sudo apt-get install apache2 libapache2-mod-php php php-mysql mysql-server 
```

**libapache2-mod-php** is providing the PHP-Module for the apache2-webserver

**php-mysql** 


## Starting the Server

```bash
sudo systemctl start apache2 
```

Once started the server will allways run when the pc is running

## Content

The content of the server will be saved in the "document root". wich depending on the apache and ubuntu version is:

`/var/www/html/`

## Initialize phpinfo

create the **phpinfo.php** in the "document root" and 

## Change Owner

```bash
 sudo chown -R www-data:www-data /var/www
 ```

## 

> /etc/apache2/sites-available/000-default.conf

```config
<Directory  /var/www/html>
    Options Indexes FollowSymLinks MultiViews
    AllowOverride All
    Order allow,deny
    Allow from all
</Directory>
```

<kbd>alt</kbd> + <kbd>t</kbd>
