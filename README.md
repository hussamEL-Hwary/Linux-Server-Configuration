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
- [Install-Required-Packages](#install-required-packages)
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
### Configure-timezone-to-UTC
### Install-apache2
### Install-PostgreSQL
### Clone-Items-Catalog
### Virtual-Environment
### Install-Required-Packages
### Create-Database
### App-Config-File
### Apache2-Config-File
### Final-step
