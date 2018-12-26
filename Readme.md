Linux Server Configuration
This is a set of instructions on how to set up a Ubuntu Linux server to host a simple web application built with Flask.
The instructions are written specifically for hosting an app called Novel on an Amazon Lightsail instance but can easily be adapted to work for an alternative application and/or server provider.
1. Details specific to the server I set up

The IP address is 52.62.64.170

The SSH port used is 2200.

The URL to the hosted webpage is:http://ec2-52-62-54-170.ap-southeast-2.compute.amazonaws.com
login by ssh -i udacity.pem -p 2200 udacity@52.62.54.170

Steps to configure linux server:
Launch your Virtual Machine with your amazon account and means we can say that with your private key.
so before that
1 Create an amazon lightsail instance of ubuntu.
->Create an instance with Amazon Lightsail

->Sign in to Amazon Lightsail using an Amazon Web Services account

->Follow the 'Create an instance' link

->Choose the 'OS Only' and 'Ubuntu 16.04 LTS' options

->Choose a payment plan

->Give the instance a unique name and click 'Create'

->Wait for the instance to start up
2.Connection to the instance:
Connect to the instance on a local machine

Note: While Amazon Lightsail provides a broswer-based connection method, this will no longer work once the SSH port is changed (see below). The following steps outline how to connect to the instance via the Terminal program on Mac OS machines (this can also be done on a Windows machine with a program such as PuTTY).

Download the instance's private key by navigating to the Amazon Lightsail 'Account page'

Click on 'Download default key'

A file called LightsailDefaultPrivateKey.pem will be downloaded; open this in a text editor

Copy the text and put it in a file called lightrail_key.rsa in the local ~/.ssh/ directory

Run chmod 600 ~/.ssh/lightrail_key.rsa

Log in with the following command: ssh -i ~/.ssh/lightrail_key.rsa ubuntu@XX.XX.XX.XX, where XX.XX.XX.XX is the public IP address of the instance (note that Lightsail will not allow someone to log in as root; ubuntu is the default user for Lightsail instances)
3.Updation
As soon as you log in virtual machine update it.
sudo apt-get update
sudo apt-get upgrade
4.Configure the firewall
 Configure the Uncomplicated Firewall (UFW)

 ->$ sudo ufw default deny incoming
 ->$ sudo ufw default allow outgoing
 ->$ sudo ufw allow 2200/tcp
 ->$ sudo ufw allow www
 ->$ sudo ufw allow ntp
 ->$ sudo ufw enable
5.Access the remote machine with the port 2200 but before that also change the connection by editing in networking part of lightsail account also .
Access the virtual machine like this.
ssh -i ~/.ssh/lightrail_key.rsa -p 2200 ubuntu@XX.XX.XX.XX, where XX.XX.XX.XX is the public IP address of the instance
6.Create a new user udacity:
sudo adduser udacity
gpasswd -s udacity sudo 
to give sudo permissions to the user.
run sudo -l 
to check that it is added or not in sudo
something like this will come.
Matching Defaults entries for grader on
    ip-XX-XX-XX-XX.ec2.internal:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User grader may run the following commands on
	ip-XX-XX-XX-XX.ec2.internal:
    (ALL : ALL) ALL
7. now give user udacity the permissions to directly log in the remote machine as user udacity.
generate key-pair with ssh-keygen
Save keygen file into (/home/user/.ssh/udacity.pem).and fill the password . 2 keys will be generated, public key (udacity.pem.pub) and identification key(udacity.pem).
Login into grader account using ssh -v udacity@"public_IP_address" -p 2200. type the password that you have fill during user creation (sudo adduser udacity step 3) .
ssh -i udacity.pem -p 2200 udacity@52.62.54.170 
if the password is correct , you will login as udacity account
make a directory in udacity account : mkdir .ssh
make a authorized_keys file using touch .ssh/authorized_keys
from your local machine,copy the contents of public key(udacity.pem.pub).
paste that contents on authorized_keys of udacity account using nano authorized_keys and save it .
give the permissions : chmod 700 .ssh and chmod 644 .ssh/authorized_keys.
do nano /etc/ssh/sshd_config , change PasswordAuthentication to no .
also make permitrootlogin no.
sudo service ssh restart.
ssh udacity@52.62.54.170 -p 2200 -i ~/.ssh/udacity.pem in new terminal .A pop-up window will open for authentication. just fill the password that you have fill during ssh-keygen creation.
8.Configure the local timezone.
 Configure local timezone to UTC

Change the timezone to UTC using following command: $ sudo timedatectl set-timezone UTC.
You can also open time configuration dialog and set it to UTC with: $ sudo dpkg-reconfigure tzdata.
Install ntp daemon ntpd for a better synchronization of the server's time over the network connection:
 $ sudo apt-get install ntp
9.Install and Configure Apache2, mod-wsgi and Git

 $ sudo apt-get install apache2 libapache2-mod-wsgi git
Enable mod_wsgi:
 $ sudo a2enmod wsgi
 as well as enable 
 sudo a2ensite 000-default.conf
10. Install and configure PostgreSQL

Installing PostgreSQL Python dependencies:
 $ sudo apt-get install libpq-dev python-dev
Installing PostgreSQL:
  $ sudo apt-get install postgresql postgresql-contrib
Check if no remote connections are allowed :
  $ sudo cat /etc/postgresql/9.3/main/pg_hba.conf
Login as postgres User (Default User), and get into PostgreSQL shell:
  $ sudo su - postgres
  $ psql
Create user catalog: CREATE USER catalog WITH PASSWORD 'password';
check lists of roles using\du
Allow the user to create database : ALTER USER catalog CREATEDB; and check the roles and attributes using \du.
Create database using : CREATE DATABASE catalog WITH OWNER catalog;
Connect to database using : \c catalog
Revoke all the rights : REVOKE ALL ON SCHEMA public FROM public;
Grant the access to catalog: GRANT ALL ON SCHEMA public TO catalog;
Once you execute database_setup.py , again you can login as psql and check all the tables with following commands:
connect to database using : \c catalog
To see the tables in schema :\dt
to see particular table:\d [tablename]
to see the entries/data in table :select * from [tablename];
to drop the table:drop table [tablename];
exit from Postgresql : \qthen exitfrom postgresql user.
restart postgresql: sudo service postgresql restart
11. Install other Packages

sudo apt-get install python-pip
source venv/bin/activate
pip install httplib2
pip install requests
sudo pip install --upgrade oauth2client
sudo pip install sqlalchemy
pip install Flask-SQLAlchemy
sudo pip install flask-seasurf
If you want to see what packages have been installed with your installer tools : pip freeze
12.clone the app from your github in /var/www
13.Make a bookCatalog.wsgi file to serve the application over the mod_wsgi. with content:

 $ touch bookCatalog.wsgi && nano bookCatalog.wsgi
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from bookCatalog import app as application
14.update the create_engine line: engine = create_engine('postgresql://catalog:catalog-pw@localhost/catalog')
in database_setup.py and all others.
15.Edit the default Virtual File with following content:

  $  sudo nano /etc/apache2/sites-available/000-default.conf
<VirtualHost *:80>
  ServerName XX.XX.XX.XX
  ServerAdmin admin@gmail.com
  WSGIScriptAlias / /var/www/catalog/bookCatalog.wsgi
  <Directory /var/www/catalog/>
      Order allow,deny
      Allow from all
  </Directory>
  Alias /static /var/www/catalog/static
  <Directory /var/www/catalog/static/>
      Order allow,deny
      Allow from all
  </Directory>
</VirtualHost>
16.Restart Apache to launch the app

 $ sudo service apache2 restart