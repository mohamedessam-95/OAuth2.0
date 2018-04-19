# Linux Server Configuration project
To visit the webpage hosted on the server go to this url :
http://ec2-35-178-86-84.eu-west-2.compute.amazonaws.com/

While reviewing use :
  Ip Address : 35.178.86.84
  SSH Port : 2200

Concerning the Grader user :
  Private key is pasted into Notes to Reviewer along with the passphrase
  public key is stored in the server in grader/home/.ssh/authorized_keys


## Create a new user named grader :

`sudo adduser grader`
`sudo cp /etc/sudoers.d/vagrant /etc/sudoers.d/grader`
`sudo nano /etc/sudoers.d/grader`
changing `vagrant ALL=(ALL:ALL) ALL` into `grader ALL=(ALL:ALL) ALL`

## Creating key-pairs for the grader user :
I generated key-pairs on local machine using ssh-keygen then saved the private key on local machine then i put the public on on the server using the following lines

	`$ su - grader`
	`$ mkdir .ssh`
	`$ touch .ssh/authorized_keys`
	`$ vim .ssh/authorized_keys`
	
Copied the public key on the local machine to this file and changed the permissions

	`$ chmod 700 .ssh`
	`$ chmod 644 .ssh/authorized_keys`
	

## Update all currently installed packages

	sudo apt-get update
	sudo apt-get upgrade

## Configuring the sshd_config :

I typed `sudo vim /etc/ssh/sshd_config`
then : changed Port 22 to Port 2200
       changed passwordauthorization to no
       changed permitrootlogin to no

## Configure the Uncomplicated Firewall (UFW)

I configured the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)

	sudo ufw allow 2200/tcp
	sudo ufw allow 80/tcp
	sudo ufw allow 123/udp
	sudo ufw enable 
 
## Configure the local timezone to UTC

I configured the time zone using `sudo dpkg-reconfigure tzdata`
then i selected UTC

## Installed and configured Apache to serve a Python mod_wsgi application
Apache `sudo apt-get install apache2`
mod_wsgi `sudo apt-get install python-setuptools libapache2-mod-wsgi`
Apache `sudo service apache2 restart`

## Install and configure PostgreSQL
1. Install PostgreSQL `sudo apt-get install postgresql`
2. Check if no remote connections are allowed `sudo vim /etc/postgresql/9.3/main/pg_hba.conf`
3. Login as user "postgres" `sudo su - postgres`
4. Get into postgreSQL shell `psql`
5. Create a new database named catalog  and create a new user named catalog in postgreSQL shell
	
	```
	postgres=# CREATE DATABASE catalog;
	postgres=# CREATE USER catalog;
	```
5. Set a password for user catalog
	
	```
	postgres=# ALTER ROLE catalog WITH PASSWORD 'password';
	```
6. Give user "catalog" permission to "catalog" application database
	
	```
	postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
	```
7. Quit postgreSQL `postgres=# \q`
8. Exit from user "postgres" 
	
	```
	exit
	```
 
## Install git, clone and setup your Catalog App project.
1. Install Git using `sudo apt-get install git`
2. Use `cd /var/www` to move to the /var/www directory 
3. Create the application directory `sudo mkdir FlaskApp`
4. Move inside this directory using `cd FlaskApp`
5. Clone the Catalog App to the virtual machine `git clone https://github.com/kongling893/Item_Catalog_UDACITY.git`
6. Rename the project's name `sudo mv ./Item_Catalog_UDACITY ./FlaskApp`
7. Move to the inner FlaskApp directory using `cd FlaskApp`
8. Rename `website.py` to `__init__.py` using `sudo mv website.py __init__.py`
9. Edit `database_setup.py`, `website.py` and `functions_helper.py` and change `engine = create_engine('sqlite:///toyshop.db')` to `engine = create_engine('postgresql://catalog:password@localhost/catalog')`
10. Install pip `sudo apt-get install python-pip`
11. Use pip to install dependencies `sudo pip install -r requirements.txt`
12. Install psycopg2 `sudo apt-get -qqy install postgresql python-psycopg2`
13. Create database schema `sudo python database_setup.py`

## Configure and Enable a New Virtual Host
1. Create FlaskApp.conf to edit: `sudo nano /etc/apache2/sites-available/FlaskApp.conf`
2. Add the following lines of code to the file to configure the virtual host. 
	
	```
	<VirtualHost *:80>
		ServerName 52.24.125.52
		ServerAdmin qiaowei8993@gmail.com
		WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
		<Directory /var/www/FlaskApp/FlaskApp/>
			Order allow,deny
			Allow from all
		</Directory>
		Alias /static /var/www/FlaskApp/FlaskApp/static
		<Directory /var/www/FlaskApp/FlaskApp/static/>
			Order allow,deny
			Allow from all
		</Directory>
		ErrorLog ${APACHE_LOG_DIR}/error.log
		LogLevel warn
		CustomLog ${APACHE_LOG_DIR}/access.log combined
	</VirtualHost>
	```
3. Enable the virtual host with the following command: `sudo a2ensite FlaskApp`

## Create the .wsgi File
1. Create the .wsgi File under /var/www/FlaskApp: 
	
	```
	cd /var/www/FlaskApp
	sudo nano flaskapp.wsgi 
	```
2. Add the following lines of code to the flaskapp.wsgi file:
	
	```
	#!/usr/bin/python
	import sys
	import logging
	logging.basicConfig(stream=sys.stderr)
	sys.path.insert(0,"/var/www/FlaskApp/")

	from FlaskApp import app as application
	application.secret_key = 'Add your secret key'
	```

## Restart Apache
1. Restart Apache `sudo service apache2 restart `

## References:
https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
