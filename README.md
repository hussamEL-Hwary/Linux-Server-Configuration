#### Linux Server Configuration<hr>
- [Overview](#overview)
- [Server-Info](#server-info)
##### Configuration steps:
- [AWS-instance](#aws-instance)
- [Create-new-user](#create-new-user)
- [Create-SSH-Key](#create-ssh-key)
- [Configure-Firewall](#configure-firewall)
- [Configure-timezone-to-UTC](#configure-timezone-to-utc)
- [Install-apache2](#install-apache2)
- [Install-PostgreSQL](#install-postgresql)
- [Clone-Items-Catalog](#clone-items-catalog)
- [Virtual-Environment](#virtual-environment)
- [Create-Database](#create-database)
- [App-Config-File](#app-config-file)
- [Apache2-Config-File](#apache2-config-file)
- [Final-step](#final-step)<hr>
### Overview
Installing  a Linux server and prepare it to host 
[Items Catalog](https://github.com/hussamEL-Hwary/Flask-items-catalog) application, securing the server from a number of attack vectors, installing and configuring a database server, and deploy Items Catalog web applications onto it.
### Server-Info
###### Address: [http://35.177.244.230/](http://35.177.244.230/)
###### SSH port: 50683
### AWS-instance
* Create an Ubuntu instance on AWS [LightSail](https://lightsail.aws.amazon.com/)
* login using instance setting page
* update ```sudo apt-get update```
* upgrade ```sudo apt-get upgrade```
### Create-new-user
* Add new user grader  ```sudo adduser grader```
* Give sudo access to user grader ```sudo nano /etc/sudoers.d/grader```
* Add following line to this file
  ```grader ALL=(ALL:ALL) ALL```
### Create-SSH-Key
##### 1- On your local machine
* generate SSH key pair with: ```ssh-keygen``` for example grader
* save your keygen file in your ssh directory ```.ssh``` 
##### 2- On your server
* login to your server and create new dirctory .ssh ```mkdir .ssh```
* Set permissions for .ssh: ```chmod 700 .ssh```
* make file to store authorized keys inside ssh directory: ```touch .ssh/authorized_keys```
* Set permissions: ```chmod 644 .ssh/authorized_keys```
* from your local machine copy the content from previously created key ```grader.pub``` back to your server
and paste it in ```.ssh/authorized_keys``` file
##### Disable password login
* Change ```PasswordAuthentication``` from ```yes```  to ```no```. ```sudo nano /etc/ssh/sshd_config```
  and save the file
##### Disable root user login
* Change ```PermitRootLogin``` to ```no```. ```sudo nano /etc/ssh/sshd_config```
  and save the file
##### Change default port to 50683
* first go to  Amazon lightsail server Head to your
  instance - > Networking -> Firewall and allow 50683/tcp custom port.
* Change ```Port``` to ```50683```. ```sudo nano /etc/ssh/sshd_config```
  and save the file
* Run  ```sudo service ssh restart``` to restart ssh service.
### Configure-Firewall
``` 
  sudo ufw default deny incoming
  sudo ufw default allow outgoing
  sudo ufw allow 50683/tcp
  sudo ufw allow www
  sudo ufw allow ntp
  sudo ufw enable
  ```
### Configure-timezone-to-UTC
* run ```sudo dpkg-reconfigure tzdata``` from prompt: 
  select ```none of the above```. Then select ```UTC```.
### Install-apache2
* ```sudo apt-get install apache2 libapache2-mod-wsgi```
* Enable mod_wsgi if not: ```sudo a2enmod wsgi```

### Install-PostgreSQL
* Installing PostgreSQL Python dependencies: ```sudo apt-get install libpq-dev python-dev```
* Installing PostgreSQL: ```sudo apt-get install postgresql postgresql-contrib```
### Clone-Items-Catalog
* Make a catalog named directory in /var/www ```sudo mkdir /var/www/catalog```
* Change the owner of the directory catalog ```sudo chown -R grader:grader /var/www/catalog```
* Clone the bookCatalog to the catalog directory: ```https://github.com/hussamEL-Hwary/Flask-items-catalog.git catalog```
* Create new branch production of repo  catalog : ```cd catalog && git checkout -b production```
* change ```app.py``` to ```__init__.py``` 
##### make .git inaccessible
* from ```cd /var/www/catalog/``` create .htaccess file ```sudo nano .htaccess```
  paste  ```RedirectMatch 404 /\.git```
### Virtual-Environment
* install python-pip
 ```sudo apt-get install python-pip``` 
* install virtualenv
 ```sudo pip install virtualenv```
* cd to catalog file ```cd /var/www/catalog/catalog``` and run
  ```
  sudo pip install virtualenv 
  sudo virtualenv venv
  source venv/bin/activate 
  ```
 * And then install app required packages
  ```
  sudo pip install Flask httplib2 
  sudo pip install oauth2client sqlalchemy sqlalchemy_utils
  sudo pip install flask-login WTForms sudo pip install requests
  sudo pip2 psycopg2  
  ```
 *  test if the installation is successful  ```python __init__.py```
 * to deactivate the environment write ```deactivate```

### Create-Database
* Login as postgres User:<br>
```sudo su - postgres```
* cd to app file<br>
```cd /var/www/catalog/catalog```
* get into PostgreSQL shell:<br>
```psql```
* Create a new User named *catalog*:<br>
```# CREATE USER catalog WITH PASSWORD 'password';```
* Create a new DB named *catalog*:<br>
```# CREATE DATABASE catalog WITH OWNER catalog;```
* Connect to the database *catalog* :<br>
```# \c catalog```
* Revoke all rights:<br>
```# REVOKE ALL ON SCHEMA public FROM public;```
* Lock down the permissions only to user *catalog*:<br>
```# GRANT ALL ON SCHEMA public TO catalog;```
* Log out from PostgreSQL:<br>
```# \q```. Then return to the *grader* user: ``` exit```
* Inside the Flask application, the database connection is now performed with:<br>
  ```engine = create_engine('postgresql://catalog:password@localhost/catalog')```
### App-Config-File
* In catalog directory ```cd /var/www/catalog``` 
*  Make a itemsCatalog.wsgi file to serve the application over the mod_wsgi.<br>
```sudo nano itemsCatalog.wsgi```
* Add  content: <br>
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog/")

from catalog import app as application
application.secret_key = 'secret_key'
```
### Apache2-Config-File
* edit apache default virtual file:<br>
```sudo nano /etc/apache2/sites-available/000-default.conf```
* add content:<br>
```<VirtualHost *:80>
                ServerName xx.xxx.xxx.xx
                ServerAdmin admin email
                WSGIScriptAlias / /var/www/catalog/itemsCatalog.wsgi
                <Directory /var/www/catalog/catalog/>
                        Order allow,deny
                        Allow from all
                </Directory>
                Alias /static /var/www/catalog/catalog/static
                <Directory /var/www/catalog/catalog/static/>
                        Order allow,deny
                        Allow from all
                </Directory>
                ErrorLog ${APACHE_LOG_DIR}/error.log
                LogLevel warn
                CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
### Final-step
