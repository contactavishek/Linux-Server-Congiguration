Linux Server Configuration:

This is the 6th project for "Full Stack Web Developer Nanodegree" on Udacity.
In this project, a Linux virtual machine needs to be configured to support the Item Catalog Application.
You can visit  http://130.211.184.179  for the website deployed.

Launch Virtual Machine:-

Instructions for SSH access to the instance:
1.	Created a Ubuntu VM on Google cloud (instance-3) with IP: 130.211.184.179 (set by Google).
2.	Created key pair using ssh-keygen on local machine in /c/Users/Avishek/.ssh/linuxCourse
3.	Created user grader VM instance-3 with: sudo adduser grader
4.	Gave sudo access to grader by: sudo usermod –a –G sudo grader
5.	Logged as grader initially as sudo login grader in the Gcloud VM
6.	Created new file .ssh/authorized_keys and copied the contents of .ssh/linuxCourse from my local machine and pasted it into .ssh/authorized_keys and saved it.
7.	Set commands:
    $ chmod 700 .ssh
    $ chmod 644 .ssh/authorized_keys

8.	Use sudo vim /etc/ssh/sshd_config and then change Port 22 to Port 2200
9.	Made firewall rule in Google instance to allow TCP:2200; TCP:80 ; UDP:123
10.	Now can use ssh to login with the grader user: ssh grader@130.211.184.179 -p 2200 -i ~/.ssh/linuxCourse

Forcing Key Based Authentication:-

Changed the following line in the file /etc/ssh/sshd_config:

PasswordAuthentication no
PermitRootLogin prohibit-password

Change timezone to UTC:

Check the timezone with the date command. This will display the current timezone after the time. If it's not UTC change it like this:
sudo timedatectl set-timezone UTC

Update all currently installed packages:

sudo apt-get update
sudo apt-get upgrade

Configuration Uncomplicated Firewall (UFW):-

Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)

sudo ufw allow 2200/tcp
sudo ufw allow 80/tcp
sudo ufw allow 123/udp
sudo ufw enable

Install and configure Apache to serve a Python mod_wsgi application:

1.	Install Apache sudo apt-get install apache2
2.	Install mod_wsgi sudo apt-get install python-setuptools libapache2-mod-wsgi
3.	Restart Apache sudo service apache2 restart

Install and configure PostgreSQL:

1.	Install PostgreSQL sudo apt-get install postgresql
2.	Check if no remote connections are allowed sudo vim /etc/postgresql/9.3/main/pg_hba.conf
3.	Login as user "postgres" sudo su - postgres
4.	Get into postgreSQL shell psql
5.	Create a new database named catalog and create a new user named catalog in postgreSQL shell
6.	postgres=# CREATE DATABASE catalog;
7.	postgres=# CREATE USER catalog;
8.	Set a password for user catalog
9.	postgres=# ALTER ROLE catalog WITH PASSWORD 'udacity';
10.	Give user "catalog" permission to "catalog" application database
11.	postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
12.	Quit postgreSQL postgres=# \q
13.	Exit from user "postgres"
    exit

Install git, clone and setup my Item Catalog project:

1.	Git was already installed.
2.	Use cd /var/www to move to the /var/www directory
3.	Created the application directory sudo mkdir FlaskApp
4.	Moved inside this directory using cd FlaskApp
5.	Created another directory mkdir FlaskApp
6.	Moved inside the inner FlaskApp directory using cd FlaskApp
7.	Clone the Item Catalog App to the virtual machine: sudo git clone https://github.com/contactavishek/Item-Catalog-Application-Project.git
8.	Renamed the project's name sudo mv ./Item-Catalog-Application-Project ./FlaskApp
9.	Moved to the innermost FlaskApp directory using cd FlaskApp
        Renamed projectlogin.py to __init__.py and edited database_setup.py, lotsofmenus.py and __init__.py files with engine = create_engine('postgresql://catalog:udacity@localhost/catalog')
        Created database schema: sudo python database_setup.py
	    Added menu items to database : sudo python lotsofmenus.py
10.	Install pip sudo apt-get install python-pip
11.	Install psycopg2 sudo apt-get -qqy install postgresql python-psycopg2

Update the Google OAuth client secrets file:

In the client_secrets.json file, added the IP address(130.211.184.179) in the "javascript_origins" field. Also added this IP address into the Google Developers Console -> API Manager -> Credentials, in the web client under "Authorized JavaScript origins"

Added the absolute path in __init__.py file by:

import os
path = os.path.dirname(__file__)
CLIENT_ID = json.loads(open(path+'/client_secrets.json', 'r').read())['web']['client_id']

Configure and Enable a New Virtual Host:

Create Flaskapp.conf to edit: sudo nano /etc/apache2/sites-available/Flaskapp.conf

Add the following lines of code to the file to configure the virtual host:

<VirtualHost *:80>
	ServerName 130.211.184.179
	ServerAdmin contactavishek239@gmail.com
	WSGIScriptAlias / /var/www/Flaskapp/FlaskApp/flaskapp.wsgi
	<Directory /var/www/FlaskApp/FlaskApp/FlaskApp/>
		Order allow,deny
		Allow from all
	</Directory>
	Alias /static /var/www/FlaskApp/FlaskApp/FlaskApp/static
	<Directory /var/www/FlaskApp/FlaskApp/FlaskApp/static/>
		Order allow,deny
		Allow from all
	</Directory>
	ErrorLog ${APACHE_LOG_DIR}/error.log
	LogLevel warn
	CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>


Enable the virtual host with the following command: sudo a2ensite Flaskapp.conf

Create the .wsgi File

1.	Create the .wsgi File under /var/www/FlaskApp/FlaskApp

    cd /var/www/FlaskApp/FlaskApp
    sudo nano flaskapp.wsgi

2.	Add the following lines of code to the flaskapp.wsgi file:
   
   #!/usr/bin/python
   import sys
   import logging
   logging.basicConfig(stream=sys.stderr)
   sys.path.insert(0,"/var/www/FlaskApp/FlaskApp")

   from FlaskAapp import app as application
   application.secret_key = 'Added client secret key from Google API '

Restart Apache:
	sudo service apache2 restart

References:
https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
