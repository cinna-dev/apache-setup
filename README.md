# Apache-setup

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