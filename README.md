# TechnicSolder
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
![Version](https://img.shields.io/badge/dynamic/json.svg?label=Version&url=http%3A%2F%2Ftgsapi.8u.cz%2Fapi%2F&query=version&colorB=blue)
![Stream](https://img.shields.io/badge/dynamic/json.svg?label=Stream&url=http%3A%2F%2Ftgsapi.8u.cz%2Fapi%2F&query=stream&colorB=yellow)

TechnicSolder is an API that sits between a modpack repository and the Technic Launcher. It allows you to easily manage multiple modpacks in one single location.

Using Solder also means your packs will download each mod individually. This means the launcher can check MD5's against each version of a mod and if it hasn't changed, use the cached version of the mod instead. What does this mean? Small incremental updates to your modpack doesn't mean redownloading the whole thing every time!

Solder also interfaces with the Technic Platform using an API key you can generate through your account there. When Solder has this key it can directly interact with your Platform account. When creating new modpacks you will be able to import any packs you have registered in your Solder install. It will also create detailed mod lists on your Platform page! (assuming you have the respective data filled out in Solder) Neat huh?

# Installation
> ***NOTE: If your server hosting provider already installed Apache Web server, you can*** [skip to step 6](https://github.com/TheGameSpider/TechnicSolder/blob/master/README.md#cloning-technicsolder-repository) <br />
> ***NOTE: If your server hosting provider already installed Apache Web server and does not support SSH access. you can just simply upload the master branch to root directory*** <br />
## Ubuntu server installation
**1. Install Ubuntu Server 18.04 (https://www.ubuntu.com/download/server)** <br />
**2. Login to Ubuntu with credentials you set.** <br />
**3. Become root**
```bash
sudo passwd root
```
Write you current password, then set new root password
```bash
exit
```
Login as root <br />
## Web server installation and configuration
**4. Install LAMP pack**<br />
Note: apt will tell you which packages it plans to install and how much extra disk space they'll take up. Press Y and hit Enter to continue, and the installation will proceed.
```bash
apt update
apt install apache2
apt install mysql-server
mysql_secure_installation
```
You will be asked if you want to configure the VALIDATE PASSWORD PLUGIN. Answer **y**<br />
Then you'll be asked to select a level of password validation. Answer **0** and set your mysql password.<br />
You'll be asked for a few more things, answer **y** to all.
```bash
apt install php libapache2-mod-php php-mysql
nano /etc/apache2/mods-enabled/dir.conf
```
We want to move the PHP index file to the first position after the DirectoryIndex specification: <br />
**DirectoryIndex index.html index.cgi index.pl index.php index.xhtml index.htm** -> **DirectoryIndex index.php**<br />
Configuration will look like this:
```
<IfModule mod_dir.c>
    DirectoryIndex index.php
</IfModule>
```
When you are finished, save and close the file by pressing Ctrl-X. You'll have to confirm the save by typing Y and then hit Enter to confirm the file save location.
```bash
service apache2 restart
nano /var/www/html/index.php
```
This will open a blank file. We want to put the following text, which is valid PHP code, inside the file:
```php
<?php
phpinfo();
```
When you are finished, save and close the file.<br />
Now we can test whether our web server can correctly display content generated by a PHP script. To try this out, we just have to visit this page in our web browser. You'll need your server's public IP address.
```bash
curl http://icanhazip.com
```
Open in your web browser: http://your_server_IP_address <br />
This page basically gives you information about your server from the perspective of PHP. It is useful for debugging and to ensure that your settings are being applied correctly.<br />
<br />
If this was successful, then your PHP is working as expected.<br />
<br />
You probably want to remove this file after this test because it could actually give information about your server to unauthorized users. To do this, you can type this:
```bash
rm /var/www/html/index.php
```
**5. Enable RewriteEngine**<br />
```bash
nano /etc/apache2/sites-enabled/000-default.conf
```
Add this before &lt;/VirtualHost&gt; close tag:
```
<Directory /var/www/TechnicSolder>
        Options Indexes FollowSymLinks MultiViews
        AllowOverride All
        Require all granted
    </Directory>
```
Save and close the file
```bash
a2enmod rewrite
```
## Cloning TechnicSolder repository
**6. Clone TechnicSolder repository** 
```bash
cd /var/www/
git clone https://github.com/TheGameSpider/TechnicSolder.git
nano /etc/apache2/sites-enabled/000-default.conf
```
Change DocumentRoot to /var/www/TechnicSolder<br />
Restart apache
```bash
service apache2 restart
```
Installation is complete. Now you need to confige TechnicSolder before using it.
# Configuration
**1. Configure MySQL**
```bash
mysql -p -u root
```
Login with your password you set earlier. <br />
Create new user
```MYSQL
CREATE USER 'solder'@'localhost' IDENTIFIED BY 'secret';
```
> **NOTE: By writing *IDENTIFIED BY 'secret'* you set your password. Dont use *secret***<br />
Create database solder and grant user *solder* access to it.
```MYSQL
CREATE DATABASE solder;
GRANT ALL ON solder.* TO 'solder'@'localhost';
exit
```
Open your TechnicSolder configuration.
```bash
nano /var/www/TechnicSolder/config.php
```
Change **db-user** to **solder**<br />
Change **db-pass** to password you set for user *solder*<br />
**2. Configure login credentials**<br />
There is only one thing to say: *Do not use the password that you are using to login to your email.*<br />
**3. Link TechnicSolder with your https://technicpack.net account**<br />
First, go to your profile and click *Edit Profile*<br />
Then click *Solder Configuration* and copy your API Key<br />
In your config.php file set api_key to key you copied.<br />
Now, you can save the config.<br />
The final step is to set your Solder URL in Solder Configuration (In your https://technicpack.net profile)
```
http://your_server_IP_address/api/
```
Click **Link solder**<br />
That's it. You have successfully installed and configured TechnicSolder. It's ready to use!
