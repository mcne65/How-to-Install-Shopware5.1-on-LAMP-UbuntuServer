# How-to-Install-Shopware5.1-on-LAMP-UbuntuServer
How to Install Shopware 5.1 on LAMP Ubuntu Server 16.04
(2017-07-21)

Hi guys! For this How to guide you'll need an Ubuntu Server running LAMP.
If you don't have LAMP (Linux, Apache, MySQL and PHP) installed then you can follow my other How-to:  
[How to Install a LAMP in Ubuntu Server 16.04](https://github.com/evedes/How-to-Install-a-LAMP-in-Ubuntu-Server)

-----------------------------------------------------------------------------------------------------------

In spite of having php 7.0 installed you might need some more php modules to run Shopware so you should install also these:

`sudo apt-get -y install php-fpm php-cli php-json php-curl php-gd php-mysql php-xml php-mbstring php7.0-zip` 

Now that you've installed the php mods we can install other stuff we'll need such as: 

[ant](http://ant.apache.org/) : `sudo apt-get install ant`




`sudo apt-get install unzip`

Set PHP memory limit to 512MB, change the values of upload_max_fliesize and post_max_size to 100M and set the timezone to UTC

``` 
sed -i "s/memory_limit = .*/memory_limit = 512M/" /etc/php/7.0/cli/php.ini
sed -i "s/;date.timezone.*/date.timezone = UTC/" /etc/php/7.0/cli/php.ini
sed -i "s/;cgi.fix_pathinfo=1/cgi.fix_pathinfo=0/" /etc/php/7.0/fpm/php.ini
sed -i "s/upload_max_filesize = .*/upload_max_filesize = 100M/" /etc/php/7.0/fpm/php.ini
sed -i "s/post_max_size = .*/post_max_size = 100M/" /etc/php/7.0/fpm/php.ini
``` 

Get the shopware installation:

`wget https://github.com/shopware/shopware/archive/v5.1.6.zip` 

Unzip file, rename folder to shopware and move it to /var/www/html/


Create Database

 `mysql -u root -p`

 ```
CREATE DATABASE shopware;
GRANT ALL PRIVILEGES ON shopware.* TO ‘shopware’@‘localhost’ IDENTIFIED BY ‘your-password’;
FLUSH PRIVILEGES;
\q
```

Go to /shopware/build

ant configure

ant build-unit

Get test images: 

wget -O test_images.zip http://releases.s3.shopware.com/test_images.zip

unzip test_images.zip

```
touch /etc/apache2/sites-available/shopware.conf
ln -s /etc/apache2/sites-available/shopware.conf /etc/apache2/sites-enabled/shopware.conf
vi /etc/apache2/sites-available/shopware.conf
``` 

```
<VirtualHost *:80>
ServerAdmin admin@mail.com
DocumentRoot "/var/www/html/shopware/"
ServerName evedes.ddns.net
ServerAlias evedes.ddns.net
<Directory "/var/www/html/shopware/">
Options FollowSymLinks
AllowOverride All 
Order allow,deny
Allow from all
</Directory>
ErrorLog /var/log/apache2/evedes.ddns.net-error_log
CustomLog /var/log/apache2/evedes.ddns.net_log common
</VirtualHost>
```

```
chown www-data:www-data -R /var/www/html/shopware/
chmod -R 777 /var/www/html/shopware/
```  


service apache2 restart ; service mysql restart ; service php7.0-fpm restart
