

#  Project: Linux Server Configuration


This page explains how to set up and secure a Linux distribution/OS on a virtual machine, install and configure a web and database server to host a web application.
 
- The Linux distribution is [Ubuntu](https://www.ubuntu.com/download/server) 16.04 LTS.
- The virtual private server is [Digital Ocean](https://digitalocean.com).
- The web application is my [Restaurant Menu CRUD App](https://github.com/saltgen/Restaurant-Menu-App-CRUD) created earlier in this Nanodegree program.
- The database server is [PostgreSQL](https://www.postgresql.org/).
- My local machine is Ubuntu 16.04.
- Google API is not accepting bare APIs anymore hence Authentication has been disabled from back-end.
- Password set for user account grader: test


we will deploy a flask server project to a Ubuntu Server(Droplet) on Digital Ocean.

We will setup an internal postgres server as the database and demonstrate some advanced server management.


## Table of Contents

1. Start a new server (or Droplet) on Digital Ocean.
2. SSH into the server (ssh grader@142.93.208.96 -p 2200 -i /home/saltgen/.ssh/grader)
3. Update all currently installed packages.

```sh
sudo apt-get update
sudo apt-get upgrade

```

4. Change the SSH port from 22 to 2200. Make sure to configure the firewall to allow it.

- Open ssh config in `nano`.

```sh
sudo nano /etc/ssh/sshd_config
```

Locate "Port 22" in that file. Change it to "Port 2200".

Restart ssh service.

```sh
sudo service ssh restart
```

5. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).

```sh
sudo ufw allow 2200
sudo ufw allow 80
sudo ufw allow 123
```

Then enable ufw.

```sh
sudo ufw enable
```
Check the status/Rules of ufw firewall

```sh
sudo ufw status
```

Now ssh port has been changed to 2200. Try exiting the ssh connection and re-connecting with the following command.

```sh
ssh grader@142.93.208.96 -p 2200 -i /home/USER_FOLDER/.ssh/grader
```

6. Create a new user account named `grader`.

```sh
sudo adduser grader
```

7. Give `grader` the permission to sudo.

To give `grader` sudo permission.

```sh
sudo usermod -aG sudo grader
```

8. Create an SSH key pair for `grader` using the ssh-keygen tool

We will now have to setup a ssh key-pair for grader.

Login as grader, ssh grader@142.93.208.96 -p 2200 -i /home/USER_FOLDER/.ssh/grader
Don't give a password this time as ssh keys are themselves meant to be credentials so you can say that they are themselves passwords.

Copy the contents of `grader.pub` file.

On the server terminal, run -

```sh
ssh grader@142.93.208.96 -p 2200 -i /home/USER_FOLDER/.ssh/grader
mkdir .ssh
chmod 700 .ssh
nano .ssh/authorized_keys
# paste the contents and save the file
chmod 644 .ssh/authorized_keys
```
Restart the ssh service.

```sh
sudo service sshd restart
```

9. Configure the local timezone to UTC.

To configure timezone, run the following command.

```sh
sudo dpkg-reconfigure tzdata
```

Select "None of the above" and then select UTC.

When done, you should see something like this in the terminal.

```sh
Current default time zone: 'Etc/UTC'
Local time is now:      Sat Jul 15 04:50:15 UTC 2017.
Universal Time is now:  Sat Jul 15 04:50:15 UTC 2017.

10. Install and configure Apache to serve a Python mod_wsgi application.

To serve Python using Apache and mod_wsgi, install the following components.

```sh
sudo apt-get install apache2 libapache2-mod-wsgi python-dev


Then start apache service.

```sh
sudo service apache2 restart
```

11. Install and configure PostgreSQL.

   - Create a new database user named `catalog` that has limited permissions to your `catalog` application database.

Install postgresql as follows

```sh
sudo apt-get install postgresql
```

Now to create `catalog` database, run the following to get into psql shell.

```sh
sudo -u postgres psql
```

## Step 12:

Install postgresql as follows

```sh
sudo apt-get install postgresql
```

Now to create `catalog` database, run the following to get into psql shell.

```sh
sudo -u postgres psql
```

Then when inside psql shell, run the following.

```sql
create user catalog with password 'password';
create database catalog with owner catalog;
```

Then exit psql shell with the following command.

```sql
\q
```


## Step 13: Restaurant Menu CRUD App config

Step 13.1: Set up and enable a virtual host

    Add the following line in /etc/apache2/mods-enabled/wsgi.conf file

    sudo nano /etc/apache2/sites-available/FlaskApp.conf and add the following lines to configure the virtual host:

<VirtualHost *:80>
                ServerName 142.93.208.96
                ServerAdmin sdnirvana94@gmail.com
                WSGIScriptAlias / /var/www/html/FlaskApp/FlaskApp/flaskapp.wsgi
                <Directory /var/www/html/FlaskApp/FlaskApp>
                        Order allow,deny
                        Allow from all
                </Directory>
                Alias /static /var/www/html/FlaskApp/FlaskApp/static
                <Directory /var/www/html/FlaskApp/FlaskApp/static/>
                        Order allow,deny
                        Allow from all
                </Directory>
                ErrorLog ${APACHE_LOG_DIR}/error.log
                LogLevel warn
                CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>


Enable virtual host: 

```sh
sudo a2ensite FlaskApp
```
Reload Apache: 
```sh
sudo service apache2 reload.
```

Step 13.2: Set up the Flask application

Create /var/www/FlaskApp/FlaskApp/flaskapp.wsgi file add the following lines:

```sh
sudo nano /var/www/FlaskApp/FlaskApp/flaskapp.wsgi
'''

 #!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/html/FlaskApp/")

from FlaskApp import app as application
application.secret_key = 'Add your secret key'


Restart Apache: sudo service apache2 restart.

Step 13.3: Set up the database schema and populate the database

python database_setup.py
python lotsofmenus.py

Step 13.4: Disable the default Apache site

Disable the default Apache site: sudo a2dissite 000-default.conf.

Reload Apache: sudo service apache2 reload.


Once this is done, enable the site and restart Apache.

```sh
sudo a2ensite FlaskApp  # enable site
sudo service apache2 reload
```

The server should be live now. Visit the IP to check (http://142.93.208.96). If an error occurs, check the logs.

```sh
sudo tail /var/log/apache2/error.log
```


## Step 14:

To disable root login & password-based login through ssh, open the ssh config file.

```sh
sudo nano /etc/ssh/sshd_config
```

Make changes as shown below.

```sh
PermitRootLogin no
PasswordAuthentication no
```

Save the file and restart ssh server.

```sh
sudo service sshd restart
```


### References

* DigitalOcean Articles

* [How To Configure the Apache Web Server on an Ubuntu or Debian VPS](https://www.digitalocean.com/community/tutorials/how-to-configure-the-apache-web-server-on-an-ubuntu-or-debian-vps)

* [How To Install the Apache Web Server on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-install-the-apache-web-server-on-ubuntu-16-04)

* Youtube - [Setting up Flask on a Digitalocean Droplet](https://www.youtube.com/watch?v=bMEAtCuFoiw)
* [Disable root login](https://mediatemple.net/community/products/dv/204643810/how-do-i-disable-ssh-login-for-the-root-user)
* Thanks to @aviaryan on GitHub for the README reference


