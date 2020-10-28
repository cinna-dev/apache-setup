# Apache-setup


## Installation

```bash
sudo apt-get install apache2
```

or install a LAMP Stack:

```bash
sudo apt-get install apache2 libapache2-mod-php php php-mysql mysql-server 
```

<https://wiki.ubuntuusers.de/LAMP/>

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

create the **phpinfo.php** in the "document root" and insert:

```php
<?php
phpinfo();
?>
```

insert `http://localhost/phpinfo` into the browser to see all information about the PHP Version and Settings.

Delete this file to make sure Outsiders can not inspect your settings.

## Change Owner

```bash
sudo chown -R www-data:www-data /var/www
```

## 000-default.conf

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


## Wordpress

### htaccess

<https://www.youtube.com/watch?v=V91ffS0yiN0&ab_channel=RankYa>

The .htaccess is a distributed configuration file, and is how Apache handles configuration changes on a per-directory basis.

WordPress uses this file to manipulate how Apache serves files from its root directory, and subdirectories thereof. Most notably, WP modifies this file to be able to handle pretty permalinks.

create a .htaccess file inside the root folder of your wordpress dir

```
# BEGIN WordPress

RewriteEngine On
RewriteBase /
RewriteRule ^index\.php$ - [L]
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule . /index.php [L]

# END WordPress
```

or if your site uses https:

```
<IfModlue mod_rewrite.c>
# BEGIN WordPress

RewriteEngine On
RewriteBase /
RewriteRule ^index\.php$ - [L]
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule . /index.php [L]
</IfModlue>

# END WordPress
```

Used for wordpress permalink access


#### Redirect

e.g.

```dot
Redirect 301 /shoop/cart/ https://wplearninglab.com/cart/
```

`Redirect <status> <path past the domain name> <where to redirect to>` 

##### redirect for all request made to http

```
RewriteRule ^(.*)$ https://%{HTTP_HOST}/$1 [R=301,L]
```

#### Charset

```
AddDefaultCharste utf-8
```

#### Keep Header alive

keep alive same connect request instead of resending each HTTP requests

```
<ifModule mod_headers.c>
Header set Connection keep-alive
</ifModlue>
```

#### Cross Origin

> Important for using Google Fonts ,  empeded Images/Videos , stylesheets, scripts iframes and some Plugins etc.

```
<IfModule mod_hearders.c>
    Header set Access-Control-Allow_origin "*"
</IfModule>
``` 

### www-data

#### Add User to www-data Group

For being able to login via FTP , edit file , upload files , create Folders and what not...

```bash
sudo usermod -a -G www-data <User>
```

confirm `<User>` is part of www-data Group

```bash
groups <User>
```