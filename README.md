# Linux-Web-Server-Configuration
The final project in Udacity's Nanodegree

## Description

### Goals of the project from Udacity instructors:
You will take a baseline installation of a Linux server and prepare it to host your web applications. You will secure your server from a number of attack vectors, install and configure a database server, and deploy one of your existing web applications onto it.

## Useful info:

IP address: 13.59.199.35

Accessible SSH port: 2200

## Step by step walkthrough

### Create a new user named grader and grant this user sudo permissions.
1. Log into the remote VM as root user through ssh: `$ ssh root@13.59.199.35`.
2. Add a new user called grader: `$ sudo adduser grader`.
3. Create a new file under the suoders directory: `$ sudo nano /etc/sudoers.d/grader`. Fill that newly created file with the following line of text: "grader ALL=(ALL:ALL) ALL", then save it.
4. In order to prevent the "sudo: unable to resolve host" error, edit the hosts:
`$ sudo nano /etc/hosts`.
- Add the host: 127.0.1.1 ip-10-20-37-65.

### Update all currently installed packages
1.
```
$ sudo apt-get update.
$ sudo apt-get upgrade.
```
2. Install finger, a utility software to check users' status: `$ apt-get install finger`.

### Configure the local timezone to UTC
1. Open time configuration dialog and set it to UTC with: `$ sudo dpkg-reconfigure tzdata`.

### Configure the key-based authentication for grader user
1. Generate an encryption key on your local machine with: `$ ssh-keygen ~/.ssh/udacity`.
2. Log into the remote VM as root user through ssh and create the following folder: `$ sudo mkdir /home/grader/.ssh`.
3. Then make this file: `$ sudo touch /home/grader/.ssh/authorized_keys`.
4. Copy the content of the udacity_key.pub file from your local machine to the /home/grader/.ssh/authorized_keys file you just created on the remote VM. Then change some permissions:
`$ sudo chmod 700 /home/grader/.ssh`.
`$ sudo chmod 644 /home/grader/.ssh/authorized_keys`.
5. Finally change the owner from root to grader: `$ sudo chown -R grader:grader /home/grader/.ssh`.
Now you are able to log into the remote VM through ssh with the following command: `$ ssh -i ~/.ssh/udacity_key.rsa grader@13.59.199.35`.

### Enforce key-based authentication
1. `$ sudo nano /etc/ssh/sshd_config`. Find the PasswordAuthentication line and edit it to `no`.
2. `$ sudo service ssh restart`.

### Change the SSH port from 22 to 2200
1. `$ sudo nano /etc/ssh/sshd_config`. Find the Port line and edit it to 2200.
2. `$ sudo service ssh restart`.

#### *Important*
Be sure to add the port 2200 to your AWS server via your AWS homepage under the "Networking" tab.
You can also go ahead and do this for port 123.

Now you are able to log into the remote VM through ssh with the following command: `$ ssh -i ~/.ssh/udacity_key.rsa -p 2200 grader@13.59.199.35`.

### Disable ssh login for root user
1. `$ sudo nano /etc/ssh/sshd_config`. Find the PermitRootLogin line and edit it to no.
2. `$ sudo service ssh restart`.

### Configure the Uncomplicated Firewall (UFW)
1. Change allowed incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
```
$ sudo ufw allow 2200/tcp.
$ sudo ufw allow 80/tcp.
$ sudo ufw allow 123/udp.
$ sudo ufw enable.
```

### Configure cron scripts to automatically manage package updates
1. Install unattended-upgrades if not already installed: `$ sudo apt-get install unattended-upgrades`.
2. To enable it, do: `$ sudo dpkg-reconfigure --priority=low unattended-upgrades`.

### Install Apache, mod_wsgi
1. `$ sudo apt-get install apache2`.
2. Install mod_wsgi with the following command: `$ sudo apt-get install libapache2-mod-wsgi python-dev`.
3. Enable mod_wsgi: `$ sudo a2enmod wsgi`.
4. `$ sudo service apache2 start`.

### Clone the Catalog app from Github
1. `$ cd /var/www`. Then: `$ sudo mkdir catalog.`
2. Change owner for the catalog folder: `$ sudo chown -R grader:grader catalog`.
3. Move inside that newly created folder and clone the catalog repository from Github: `$ git clone https://github.com/dwdewul/item-catalog.git catalog`.
4. Make a catalog.wsgi file to serve the application over the mod_wsgi.
```python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application
```

### Install virtual environment, Flask and the project's dependencies

1. Install pip: `$ sudo apt-get install python-pip`.
2. Install Virtualenv: `$ sudo pip install virtualenv`.
3. Move to the catalog folder: `$ cd /var/www/catalog`. 
4. Create a new virtual environment with the following command: `$ sudo virtualenv venv`.
5. Activate the virtual environment: `$ source venv/bin/activate`.
6. Change permissions to the virtual environment folder: `$ sudo chmod -R 777 venv`.
7. Install Flask: `$ pip install flask`.
8. Install all the other project's dependencies: `$ pip install [insert package here] ` 
#### The libraries:
- bleach 
- httplib2 
- request 
- oauth2client 
- sqlalchemy 
- python-psycopg2.

### Configure and enable a new virtual host

1. Create a virtual host conifg file: `$ sudo nano /etc/apache2/sites-available/catalog.conf`.
2. Paste in the following lines of code:
```xml
<VirtualHost *:80>
    ServerName 13.59.199.35
    ServerAdmin admin@13.59.199.35
    WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
    WSGIProcessGroup catalog
    WSGIScriptAlias / /var/www/catalog/catalog.wsgi
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
3. Enable the new virtual host: `$ sudo a2ensite catalog`.

### Install and configure PostgreSQL

1. Install some necessary Python packages for working with PostgreSQL: `$ sudo apt-get install libpq-dev python-dev`.
2. Install PostgreSQL: `$ sudo apt-get install postgresql postgresql-contrib`.
3. Postgres is automatically creating a new user during its installation, whose name is 'postgres'. That is a tusted user who can access the database software. So let's change the user with: `$ sudo su - postgres`, then connect to the database system with `$ psql`.
4. Create a new user called 'catalog' with a password:  `CREATE USER catalog WITH PASSWORD 'password';`.
5. Give catalog user the CREATEDB capability: `ALTER USER catalog CREATEDB;`.
6. Create the 'catalog' database owned by catalog user: `CREATE DATABASE catalog WITH OWNER catalog;`.
7. Connect to the database: `\c catalog`.
8. Revoke all rights: `REVOKE ALL ON SCHEMA public FROM public;`.
9. Lock down the permissions to only let catalog role create tables: `GRANT ALL ON SCHEMA public TO catalog;`.
10. Log out from PostgreSQL: `\q`. Then return to the grader user: `$ exit`.
11. Inside the Flask application, the database connection is now performed with:
`engine = create_engine('postgresql://catalog:password@localhost/catalog')`
12. Setup the database with: `$ python /var/www/catalog/catalog/db_setup.py`.
13. To prevent potential attacks check that no remote connections to the database are allowed. Open the following file: `$ sudo nano /etc/postgresql/9.3/main/pg_hba.conf` and edit it, if necessary, to make it look like this:
```
local   all             postgres                                peer
local   all             all                                     peer
host    all             all             127.0.0.1/32            md5
host    all             all             ::1/128                 md5
```

### Update OAuth authorized JavaScript origins
1. To let users correctly log-in change the authorized URI to 13.59.199.35 on the Google developer console.

### Restart Apache to launch the app
1. `$ sudo service apache2 restart`.

HUGE thanks to [iliketomatoes](https://github.com/iliketomatoes/) and [rrjoson](https://github.com/rrjoson/) who wrote really helpful READMEs.
