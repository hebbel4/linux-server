# linux-server

These are the steps to secure your server from a number of attack vectors, install and configure a database server, and deploy one of your existing web applications onto it.

## User Info
* Public IP address: 54.160.215.248
* SSH port: 2200
* URL to my hosted web application: 54.160.215.248

## Update and upgrade all currently installed packages
* ```sudo apt-get update```
* ```sudo apt-get upgrade```
* ```sudo apt-get autoremove```

## Create new user grader
* ```sudo adduser grader```
* Enter password and other information

## Give grader user sudo access
* Copy a new file for grader: ```sudo cp /etc/sudoers.d/ubuntu /etc/sudoers.d/grader```
* Edit this file to give user access: ```sudo nano /etc/sudoers.d/grader```
* Change "root" to "grader" and save

## Generate key-pair and disable password-base login:
* On local terminal run command: ```ssh-keygen``` and specify location for key-pair
* On server: Make a new directory: ```mkdir .ssh```
* ```touch .ssh/authorized_keys```
* Copy and paste public key to authorized_keys
* Run command to set access permission: ```chmod 700 .ssh```
* Run command: ```chmod 644 .ssh/authorized_keys```

## Change port from 22 to 2200
* ```sudo ufw default deny incoming```
* ```sudo ufw default allow outgoing```
* Allow incoming TCP packets on port 2200 (SSH): ```sudo ufw allow 2200/tcp```
* Allow incoming TCP packets on port 80 (HTTP): ```sudo ufw allow www```
* Allow incoming UDP packets on port 123 (NTP): ```sudo ufw allow ntp```

## Configure the local timezone to UTC:
* Run command ```date``` to see what is your timezone, mine is UTC as default

## Install and configure Apache to serve a Python mod_wsgi application
* Install Apache web server: ```sudo apt-get install apache2```
* Install mod_wsgi for serving Python apps from Apache and the helper package python-setuptools: ```sudo apt-get install python-setuptools libapache2-mod-wsgi```
* Enable mod_wsgi: ```sudo a2enmod wsgi```

## Install Git
* ```sudo apt-get install git```
* Configure your username: ```git config --global user.name <username>```
* Configure your email: ```git config --global user.email <email>```

## Clone the Catalog app from Github
* Go to var/www directory: ```cd /var/www```
* ```git clone https://github.com/hebbel4/catalog-app.git```
* Make a catalog.wsgi file to serve the application over the mod_wsgi. That file should look like this:
```
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application
application.secret_key = 'super_secret_key'
```

## Install virtual environment, Flask and the project's dependencies
* Install pip, the tool for installing Python packages: ```sudo apt-get install python-pip```
* Install virtualenv: ```sudo pip install virtualenv```
* Activate the virtual environment: ```source venv/bin/activate```
* Install Flask: ```pip install Flask```
* Install all the other project's dependencies: ```sudo -H pip install bleach httplib2 request oauth2client sqlalchemy python-psycopg2```

# Configure and enable a new virtual host
* Create a virtual host conifg file: ```sudo nano /etc/apache2/sites-available/catalog.conf```
* The file should look like this: 
```
<VirtualHost *:80>
    ServerName 54.160.215.248
    ServerAlias ec2-54-160-215-248.compute-1.amazonaws.com
    ServerAdmin admin@54.160.215.248
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
* Enable the new virtual host: ```sudo a2ensite catalog```

## Install and configure PostgreSQL
* Install PostgreSQL: ```sudo apt-get install postgresql postgresql-contrib```
* Create a new user called 'catalog' with his password: ```# CREATE USER catalog WITH PASSWORD 'somepassword'```
* Create the 'catalog' database owned by catalog user: ```sudo -u postgres creatdb -O catalog catalog;```
* If you need to change user catalog's password: ```sudo -u postgres psql```
* Run command: ```alter user catalog with password 'catalog';```
* Inside the Flask application, the database connection is now performed with: engine = create_engine('postgresql://catalog:somepassword@localhost/catalog')
* Setup the database with: ```python /var/www/catalog/catalog/setup_database.py```

##  Restart Apache to launch the app
* ```sudo service apache2 restart```

