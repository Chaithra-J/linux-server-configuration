# Linux Server Configuration:
## Project Overview:
You will take a baseline installation of a Linux server and prepare it to host your web applications. You will secure your server from a number of attack vectors, install and configure a database server, and deploy one of your existing web applications onto it.

A deep understanding of exactly what your web applications are doing, how they are hosted, and the interactions between multiple systems are what define you as a Full Stack Web Developer. In this project, you’ll be responsible for turning a brand-new, bare bones, Linux server into the secure and efficient web application host your applications need.


## Prerequsits:
To complete this project, you'll need a Linux server instance. We recommend using Amazon Lightsail for this. If you don't already have an Amazon Web Services account, you'll need to set one up. Once you've done that, here are the steps to complete this project.


## Link to project:[Item Catalog](https://github.com/Chaithra-J/Item-catalogs.git catalog)
* Public IP:
* ssh port: 2200
* Hosted website: [http://34.207.77.81](http://34.207.77.81), [http://ec2-13-232-247-250.ap-south-1.compute.amazonaws.com](http://ec2-13-232-247-250.ap-south-1.compute.amazonaws.com)
* IP address: 34.207.77.81

## Software Installed:
* Apache2
* mod_wsgi
* PostgreSQL
* git
* pip
* virtualenv
* httplib2
* Python Requests
* oauth2client
* SQLAlchemy
* Flask
* libpq-dev
* Psycopg2
* Feedparser




## Steps to be followed:
Get your server.
1. Start a new Ubuntu Linux server instance on Amazon Lightsail.
  1. Log in.
  1. Create an instance.
  1. Choose an instance image: Ubuntu 18.04 LTS
  1. Choose your instance plan.
  1. Give your instance a hostname.
  1. Wait for it to start up.
  1. Create a static IP instead of Public IP.
  
1. Follow the instructions provided to SSH into your server:
  1. Once your instance starts up, you can log into it with SSH from your browser.
    * Download the instance's default private key from the 'Accounts Page'
    * A 'LightsailDefaultPrivateKey.pem' file gets downloaded. Open it in a text editor and save it as 'Udacity_key.rsa' (Used in this project) in ```~/.ssh/``` directory.
    * Run ```chmod 600 ~/.ssh/Udacity_key.rsa```
    * Log in with the following command: ```ssh -i ~/.ssh/Udacity_key.rsa ubuntu@XX.XX.XX.XX```, where XX.XX.XX.XX is the public IP address of the instance (note that Lightsail will not allow someone to log in as root; ubuntu is the default user for Lightsail instances).
    
  1. When you SSH in, you'll be logged as the ```ubuntu``` user. When you want to execute commands as ```root```, you'll need to use the ```sudo``` command to do it.


### Secure your server:
1. Update all currently installed packages. Notify the system of what package updates are available by running ```sudo apt-get update```
Download available package updates by running ```sudo apt-get upgrade```

1. **Change the SSH port from 22 to 2200. Make sure to configure the Lightsail firewall to allow it** 
  1. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
  1. Start by changing the SSH port from 22 to 2200 ( ```cd /etc/ssh/sshd_config```. In the sshd_config file, add Port 2200, then restart SSH by running sudo service ssh restart; restarting SSH is a very important step!)
  1. Check to see if the ufw (the preinstalled ubuntu firewall) is active by running ```sudo ufw``` status
  1. Run ```sudo ufw default deny incoming``` to set the ufw firewall to block everything coming in.
  1. Run ```sudo ufw default allow outgoing``` to set the ufw firewall to allow everything outgoing.
  1. Run ```sudo ufw allow ssh``` to set the ufw firewall to allow SSH.
  1. Run ```sudo ufw allow 2200/tcp``` to allow all tcp connections for port 2200 so that SSH starts working.
  1. Run ```sudo ufw allow www``` to set the ufw firewall to allow a basic HTTP server.
  1. Run ```sudo ufw allow 123/udp``` to set the ufw firewall to allow NTP.
  1. Run ```sudo ufw deny 22``` to deny port 22.
  1. Run ```sudo ufw``` enable to enable the ufw firewall.
  1. Run ```sudo ufw status``` to check which ports are open and to see if the ufw is active; if done correctly, it should look like this: 22(Action:DENY, From:Anywhere), 2200/tcp(Action:Allow, From:Anywhere), 80/tcp(Action:ALLOW, From:Anywhere), 123/udp(Aciton:ALLOW, From:Anythere).
  1. Update the external (Amazon Lightsail) firewall on the browser by clicking on the 'Manage' option, then the 'Networking' tab, and then changing the firewall configuration to match the internal firewall settings above (only ports 80(TCP), 123(UDP), and 2200(TCP) should be allowed; make sure to deny the default port 22).
  1. Login in on Windows, open a terminal and run ```ssh -i ~/.ssh/lightrail_key.rsa -p 2200 ubuntu@XX.XX.XX.XX```, where XX.XX.XX.XX is the public IP address of the instance. Connecting to the instance through a browser now no longer works; this is because Lightsail's browser-based SSH access only works through port 22, which is now denied.

1. **Give grader access**
  * **Create a new user account named grader** 
  1. Run ```sudo adduser grader```
  1. Enter in a new UNIX password (twice) when prompted.
  1. Fill out information for the new grader user or press enter to leave it empty.
  1. To switch to the grader user, run su - grader, and enter the password
  * **Give grader the permission to sudo** 
  1. Run ```sudo visudo```
  1. Add the following line below this one:
     ```grader	ALL=(ALL:ALL) ALL```
  1. Save and close the visudo file.
  1. To verify that grader has sudo permissions, ```run su - grader```, enter the password, and run ```sudo -l```; after entering in the password (again), a line like the following should appear, meaning grader has sudo permissions:
  
 
  Matching Defaults entries for grader on
    ip-XX-XX-XX-XX.ec2.internal:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

  User grader may run the following commands on
	  ip-XX-XX-XX-XX.ec2.internal:
    (ALL : ALL) ALL
    
  
  * **Create an SSH key pair for grader using the ssh-keygen tool**
  1. Run ```ssh-keygen``` on the local machine.
  1. Choose a file name for the key pair (Eg:Udacity_project) or press enter to save it as the default id_rsa file. This project uses 'id_rsa'.
  1. Enter in a passphrase twice (a private and a public(.pub) key are generadet).
  1. Log in to the virtual machine
  1. Switch to grader's home directory, and create a new directory. Run ```mkdir .ssh```
  1. Run ```touch .ssh/authorized_keys```
  1. On the local machine, run ```cat ~/.ssh/insert-name-of-file.pub```('id_rsa.pub' is used here).
  1. Copy the contents of the file, and paste them in the '.ssh/authorized_keys' file on the virtual machine using ```sudo nano .ssh/authorized_keys```
  1. Run ```chmod 700 .ssh``` on the virtual machine.
  1. Run ```chmod 644 .ssh/authorized_keys``` on the virtual machine.
  1. Make sure key-based authentication is forced. Log in as grader, open the '/etc/ssh/sshd_config file' using ```cd /etc/ssh/sshd_config```, and change the  from 'PasswordAuthentication yes' to 'no'. Save and exit the file. Run ```sudo service ssh restart```
  1. Log in as the grader using the following command:

  ```ssh -i ~/.ssh/grader_key -p 2200 grader@XX.XX.XX.XX```. This will ask for grader password.
  **Configure local time zone**
  1.Run ```sudo dpkg-reconfigure tzdata```, and set your location.(Here the location is set to Asia/Kolkata)
  Current default time zone: 'Asia/Kolkata'
Local time is now:      Wed Nov  7 22:07:58 IST 2018.
Universal Time is now:  Wed Nov  7 16:37:58 UTC 2018.
  1. Run ```sudo timedatectl set-ntp on```
  1. Run ```sudo timedatectl set-ntp no```
  1. Test to make sure the timezone is configured correctly by runningdate by running ```timedatectl```. The display looks like the following:
  Local time: Wed 2018-11-07 22:21:13 IST
                  Universal time: Wed 2018-11-07 16:51:13 UTC
                        RTC time: Wed 2018-11-07 16:51:14
                       Time zone: Asia/Kolkata (IST, +0530)
       System clock synchronized: yes
systemd-timesyncd.service active: no
                 RTC in local TZ: no

  ##Install and configure Apache:
  1. Install the mod_wsgi package (which is a tool that allows Apache to serve Flask applications) along with python-dev (a package with header files required when building Python extensions); use the following command:

  ```sudo apt-get install libapache2-mod-wsgi python-dev```
  1. Make sure 'mod_wsgi' is enabled by running ```sudo a2enmod wsgi```

  **Install PostgreSQL**
  1. Install PostgreSQL by running ```sudo apt-get install postgresql```
  1. Open the '/etc/postgresql/9.5/main/pg_hba.conf' file using ```sudo nano /etc/postgresql/9.5/main/pg_hba.conf```
  
  local   all             postgres                                peer
local   all             all                                     peer
host    all             all             127.0.0.1/32            md5
host    all             all             ::1/128                 md5

  * Check whether python is installed by running ```python```
  
  **Create a new PostgreSQL user**
  1. Create a new PostgreSQL user named catalog with limited permissions.
  1. PostgreSQL creates a Linux user with the name postgres during installation; switch to this user by running ```sudo su - postgres``` 
  1. Run ```psql``` command to connect to psql.
  1. Create the catalog user by running ```CREATE ROLE catalog WITH LOGIN;```
  1. Give the catalog user the ability to create databases: ```ALTER ROLE catalog CREATEDB;```
  1. Give the catalog user a password by running ```\password catalog```
  1. Check to make sure the catalog user was created by running ```\du;``` a table will be returned.
  1. Exit psql by running ```\q```
  1. Switch back to the ubuntu user by running ```exit```
  
  **Install git and clone the project**
  1. Run ```sudo apt-get install git```
  1. Change directory to /var/www/ directory by ```cd /var/www/``` command and create a FlaskApp directory by ```mkdir FlaskApp``` command.
  1. Change to the 'FlaskApp' directory by ```cd FlaskApp``` and clone the Itemcatalog project:

  ```sudo git clone https://github.com/Chaithra-J/Item-catalogs.git FlaskApp```

  1. Now the repository is stored in a new FlaskApp directory.
  1. Change the ownership of the 'FlaskApp' directory to ubuntu by running (while in /var/www):

  ```sudo chown -R ubuntu:ubuntu FlaskApp/```

  1. Change to the /var/www/FlaskApp/FlaskApp directory
  1. Change the name of the app.py file to __init__.py by running ```mv app.py __init__.py```
  1. In __init__.py(```sudo nano python __init__.py```), change:

  ```app.run(host='0.0.0.0', port=8000)```

  to:

  ```app.run()```

  **Setup a virtual environment**
  1. Install pip if it is not already installed by using ```sudo apt-get install python-pip``` command.
  1. Install virtualenv with apt-get by running ```sudo pip install virtualenv```.
  1. Change to the /var/www/FlaskApp/FlaskApp/ directory and run ```sudo virtualenv venv``` (vene is the name of the virtual enviranment used for the project)
  1. Now, install Flask in that environment by activating the virtual environment with the following command:
  ```source venv/bin/activate``` 
  1. Give this command to install Flask inside:
  ```sudo pip install Flask``` 
  1. Next, run the following command to test if the installation is successful and the app is running:
  ```sudo python __init__.py``` 
  It should display “Running on http://localhost:5000/” or "Running on http://127.0.0.1:5000/". If you see this message, you have successfully configured the app.
  1. To deactivate the environment, give the following command:
  ```deactivate```
  
  **Configure and Enable a New Virtual Host**
  1. Issue the following command in your terminal:

  1. Add the following lines of code to the file to configure the virtual host. Be sure to change the ServerName to your domain or cloud server's IP address:
  ```sudo nano /etc/apache2/sites-available/FlaskApp.conf```
  <VirtualHost *:80>
   ServerName 34.207.77.81
   ServerAlias 34.207.77.81.xip.io
   ServerAdmin youremail@domain.com
   WSGIScriptAlias / /var/www/FlaskApp/FlaskApp/flaskapp.wsgi
   <Directory /var/www/FlaskApp/FlaskApp/>
       Require all granted
   </Directory>
   Alias /static /var/www/FlaskApp/FlaskApp/static
   <Directory /var/www/FlaskApp/FlaskApp/static/>
       Require all granted
   </Directory>
   ErrorLog ${APACHE_LOG_DIR}/error.log
   LogLevel warn
   CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
  
  **Enable the virtual host**
  1. Run ```sudo a2ensite FlaskApp```
  1. Restart Apache server:
  ```sudo service apache2 restart```
  
  **Creating the .wsgi File**
  1. Apache uses the .wsgi file to serve the Flask app. Move to the /var/www/FlaskApp/ directory and create a file named flaskapp.wsgi with following commands:
  ```cd /var/www/FlaskApp/FlaskApp/```
  ```sudo nano flaskapp.wsgi```
  1. Add the following lines to the flaskapp.wsgi file:

import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/FlaskApp/FlaskApp/")

from FlaskApp import app as application

  1. Restart Apache server:
 ```sudo service apache2 restart```
  
Now you should be able to run the application at [http://34.207.77.81.xip.io/](http://34.207.77.81.xip.io/)

Note: You might still see the default Apache page despite setting everything up correctly. To resolve it and see your Flask app running, run the following commands in order:

```sudo a2dissite 000-default.conf```
```sudo service apache2 restart```

 # Sources
* Udacity course: Configuring Linux Web Servers
* Udacity course: Linux Command Line Basics
* Digital Ocean tutorial: How To Deploy a Flask Application on an Ubuntu VPS
* Digital Ocean tutorial: How To Use Roles and Manage Grant Permissions in PostgreSQL on a VPS
* Digital Ocean tutorial: How To Secure PostgreSQL on an Ubuntu VPS
* Kill the Yak tutorial for using PostgreSQL with Flask or Django
* The Hitchhiker’s Guide to Python guide on virtualenv
* Flask documentation on virtualenv
* tutorialspoint tutorial on creating a database with PostgreSQL
* Digital Ocean tutorial: An Introduction to Linux Permissions
*tutorialspoint tutorial on how to drop a PostgeSQL database
* SQLAlchemy documentation on cascade delete
* SQLAlchemy documentation on one-to-many relationships
* Flask documenation on making a virtualenv work with mod_wsgi




