# Linux Server Configuration
----
by Jasmin Shah, for the purpose of completing seventh lesson of:
[Full-Stack Web Developer Nanodegree](https://www.udacity.com/course/nd004)

# About This project
----
This project takes a baseline installation of a Linux distribution on a virtual machine and prepares it to host the [item catalog](https://github.com/Jasmin25/Project7) web application.

# Server information
----
- URL: http://52.211.246.52.xip.io
- IP address: 52.211.246.52
- SSH port: 2200

# How to run?
----
To ssh into server:
- Open Terminal or cmd
- Run `ssh -i <key_information> -p 2200 <user_name>@52.211.246.52`

# Steps for server setup
----
1. Create new user named grader and give it the permission to sudo
  - SSH into the server through `ssh -i ~/.ssh/ubuntu_key.pem ubuntu@52.211.246.52`
  - Run `$ sudo adduser grader` to create a new user named grader
  - Create a new file in the sudoers directory with `sudo nano /etc/sudoers.d/grader`
  - Add the following text `grader ALL=(ALL) NOPASSWD:ALL`
   
2. Update all packages currently installed
  - Download package lists with `sudo apt-get update`
  - Fetch new versions of packages with `sudo apt-get upgrade`

3. Change SSH port from 22 to 2200
  - Run `sudo nano /etc/ssh/sshd_config`
  - Change the port from 22 to 2200
  - Confirm by running `ssh -i ~/.ssh/udacity_key.pem -p 2200 ubuntu@52.211.246.52`
  
4. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
  - `sudo ufw allow 2200/tcp`
  - `sudo ufw allow 80/tcp`
  - `sudo ufw allow 123/udp`
  - `sudo ufw enable`
  
5. Configure the local timezone to UTC
  - Run `sudo dpkg-reconfigure tzdata` -> Choose None of the above -> Choose UTC
 
6. Configure key-based authentication for grader
  - Login as grader, and run the following
    - mkdir .ssh
    - touch .ssh/authorized_keys 
    - nano .ssh/authorized_keys
  - Copy content of `cat .ssh/grader_key.pub` from local machine to the server
  - Limit access to keys by changing permissions of the folders:
    - chmod 700 .ssh
    - chmod 644 .ssh/authorized_keys
  - Login using public keys with `ssh grader@52.211.246.52 -p 2200 -i ~/.ssh/grader+key`

7. Disable ssh login for root user and force public key based login
  - Run `sudo nano /etc/ssh/sshd_config`
  - Change `PermitRootLogin without-password` line to `PermitRootLogin no`
  - Change `PasswordAuthentication` line to `no`
  - Restart ssh with `sudo service ssh restart`
  - Now you can only login using `ssh grader@52.211.246.52 -p 2200 -i ~/.ssh/grader_key`
 
8. Install Apache
  - `sudo apt-get install apache2`

9. Install mod_wsgi
  - Run `sudo apt-get install libapache2-mod-wsgi python-dev`
  - Enable mod_wsgi with `sudo a2enmod wsgi`
  - Start the web server with `sudo service apache2 start`
  
10. Clone the Catalog app from Github
  - Install git using: `sudo apt-get install git`
  - `cd /var/www`
  - `sudo mkdir catalog`
  - Change owner of the newly created catalog folder `sudo chown -R grader:grader catalog`
  - `cd catalog`
  - Clone [this](https://github.com/Jasmin25/Project5) project from github
  - Create a catalog.wsgi file, and add the following:
  ```
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0, "/var/www/catalog/")
  
  from catalog import app as application
  application.secret_key = 'my_secret_key'
  ```
  - Rename web_app.py to __init__.py using `mv web_app.py __init__.py`
  
11. Install virtual environment
  - Install pip using `sudo apt-get install python-pip`
  - Install the virtual environment `sudo pip install virtualenv`
  - Create a new virtual environment with `sudo virtualenv venv`
  - Activate the virutal environment `source venv/bin/activate`
  - Change permissions `sudo chmod -R 777 venv`

12. In the virtual environment, install all dependencies using
 `sudo pip install flask httplib2 oauth2client sqlalchemy psycopg2 sqlalchemy_utils requests`

13. Update path of client_secrets.json file
  - `nano __init__.py`
  - Change client_secret.json path to `/var/www/catalog/catalog/client_secret.json`
  
14. Configure and enable a new virtual host
  - Run this: `sudo nano /etc/apache2/sites-available/catalog.conf`
  - Paste this code: 
  ```
  <VirtualHost *:80>
      ServerName 52.211.246.52.xip.io
      ServerAlias 52.211.246.52.xip.io
      ServerAdmin admin@52.211.246.52
      WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/catalog/venv/lib/python2.7/site-packages
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
  - Enable the virtual host `sudo a2ensite catalog`

15. Install and configure PostgreSQL using:
    - `sudo apt-get install libpq-dev python-dev`
    - `sudo apt-get install postgresql postgresql-contrib`
  - Log into Postgres DB using `sudo su - postgres`
  - Go into console mode using `psql` and type the following commands:
    - `CREATE USER catalog WITH PASSWORD 'password';`
    - `ALTER USER catalog CREATEDB;`
    - `CREATE DATABASE gameshop WITH OWNER catalog;`
    - `\c gameshop`
    - `REVOKE ALL ON SCHEMA public FROM public;`
    - `GRANT ALL ON SCHEMA public TO catalog;`
    - `\q`
    - `exit`
  - Change engine binding in `__init__.py`, `new_database_setup.py` and `populate_db.py` files to: 
  `engine = create_engine('postgresql://catalog:password@localhost/gameshop')`
  - Next, check if the contents of the file `sudo nano /etc/postgresql/9.5/main/pg_hba.conf` look like this:
  ```
  local   all             postgres                                peer
  local   all             all                                     peer
  host    all             all             127.0.0.1/32            md5
  host    all             all             ::1/128                 md5
  ```
  
16. Restart Apache using `sudo service apache2 restart`
  
17. Web app available at [http://52.211.246.52.xip.io](http://52.211.246.52.xip.io)

# License
----
The content of this repository is licensed under [MIT License](https://opensource.org/licenses/MIT)
