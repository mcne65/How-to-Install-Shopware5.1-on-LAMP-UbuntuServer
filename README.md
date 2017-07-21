# How-to-Install-Shopware5.1-on-LAMP-UbuntuServer
How to Install Shopware 5.1 on LAMP Ubuntu Server 16.04
(2017-07-21)

Hi guys! For this How to guide you'll need an Ubuntu Server running LAMP.
If you don't have LAMP (Linux, Apache, MySQL and PHP) installed then you can follow my other How-to:  
[How to Install a LAMP in Ubuntu Server 16.04](https://github.com/evedes/How-to-Install-a-LAMP-in-Ubuntu-Server)

--------------------------------------------------------------------------------------------------------

### 01. Prepare the environment to install Shopware 5

Start by doing a: `sudo -i` cause you'll need root privileges for the steps below.

In spite of having php 7.0 installed you might need some more php modules to run Shopware so you should install also these:

`sudo apt-get -y install php-fpm php-cli php-json php-curl php-gd php-mysql php-xml php-mbstring php7.0-zip` 

Set PHP memory limit to 512MB, change the values of upload_max_fliesize and post_max_size to 100M and set the timezone to UTC :

``` 
sed -i "s/memory_limit = .*/memory_limit = 512M/" /etc/php/7.0/cli/php.ini
sed -i "s/;date.timezone.*/date.timezone = UTC/" /etc/php/7.0/cli/php.ini
sed -i "s/;cgi.fix_pathinfo=1/cgi.fix_pathinfo=0/" /etc/php/7.0/fpm/php.ini
sed -i "s/upload_max_filesize = .*/upload_max_filesize = 100M/" /etc/php/7.0/fpm/php.ini
sed -i "s/post_max_size = .*/post_max_size = 100M/" /etc/php/7.0/fpm/php.ini
``` 

Now that you've installed the php mods we can install other stuff we'll need such as: 

[ant](http://ant.apache.org/) : `sudo apt-get install ant`
unzip : `sudo apt-get install unzip`

Done? Okay, let's go!

### 02. Installing Shopware 5 (Part 1 - Download and Placement)

Position yourself on root home: `cd ~` (I'm supposing you did the `sudo -i`)

Get the shopware installation:

`wget https://github.com/shopware/shopware/archive/v5.1.6.zip` 
(I've chosen this version because I think it's stable! I've tried with 5.2.7 and had some problems installing and configuring Shopware)

Unzip file: `unzip v5.1.6.zip`

Now if you do an ls you can see a directory called shopware-5.1.6

Let's call it shopware: `mv shopware-5.1.6 shopware`

Now let's move it to our webserver: `mv shopware /var/www/html` 

NOTE: If you have any index file on the server root you should delete it because it'll interfere with shopware deployment later.

### 03. Creating the Database

Create Database

`mysql -u root -p`

 ```
CREATE DATABASE shopware;
GRANT ALL PRIVILEGES ON shopware.* TO ‘shopware’@‘localhost’ IDENTIFIED BY ‘your-password’;
FLUSH PRIVILEGES;
\q
```

Basically what we've did was to create a database called **shopware** and grant privilleges on the dbuser **shopware**, located in the **localhost**, to it.

### 04. Installing Shopware 5 (Part 2 - Configuring and Building)

Go to the shopware/build directory and do: `ant configure`  

It will ask you for some configs: 

```
db-host: localhost
db-port: 3306
db-name: shopware
db-username: shopware
db-password: your-password
app.host: 192.168.1.220 (here you but the URL or the IP of the host)
app.path: /var/www/html/shopware
```

After this you need to build-unit: 

`ant build-unit` 

NOTE: In the end of the build you should see BUILD SUCCESSFUL and the Total Time it took. If you don't see this something is wrong, you should do the **ant configure** again or check your LAMP install.

Okay, if you're fine till here now we need to get some test images for our shop:

Leave the build directory: `cd ..` and place yourself in the /shopware directory.

`wget -O test_images.zip http://releases.s3.shopware.com/test_images.zip`

Now let's unzip it: `unzip test_images.zip`

Okay, everything is in place right now.

Let's create an apache2 shopware.conf (hey experienced guys, comment or do some pull request on this)

```
touch /etc/apache2/sites-available/shopware.conf
ln -s /etc/apache2/sites-available/shopware.conf /etc/apache2/sites-enabled/shopware.conf
vi /etc/apache2/sites-available/shopware.conf
``` 

Put this config inside shopware.conf:

```
<VirtualHost *:80>
ServerAdmin admin@mail.com
DocumentRoot "/var/www/html/shopware/"
ServerName yourdomain-or-ip
ServerAlias yourdomain-or-ip
<Directory "/var/www/html/shopware/">
Options FollowSymLinks
AllowOverride All 
Order allow,deny
Allow from all
</Directory>
ErrorLog /var/log/apache2/yourdomain-or-ip-error_log
CustomLog /var/log/apache2/yourdomain-or-ip_log common
</VirtualHost>
```

save and quit using `:wq`

Now let's set permissions (hey guys! comment or PR on this please!)

```
chown www-data:www-data -R /var/www/html/shopware/
chmod -R 777 /var/www/html/shopware/
```  

Finally restart these services in order for everything above to be updated:

`service apache2 restart ; service mysql restart ; service php7.0-fpm restart` 

