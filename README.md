# Linux-Server-Configuration

## Introduction
This project involves taking a baseline installation of Linux Server and preparing it to host web applications. This includes securing the server a number of attack vectors, install and configure a database server, and deploy Item Catalog app onto it.

## Server Infoormation
- Public IP : 18.130.245.167
- SSH Port : 2200

## Get your server
### Start a new Ubuntu Linux server instance on Amazon Lightsail.
- Login into [Amazon Lightsail](https://lightsail.aws.amazon.com/ls/webapp/home/resources) using your Amazon Web Services account. If you don't already have an Amazon Web Services account, you'll be prompted to create one.
- Once you are login into, `Create instance`. 
- Select `Linux/Unix` platform, Select `OS Only` blueprint and Choose `Ubuntu 16.04 LTS`.
- Choose a instance plan.
- Give your instance a hostname.
- Then click `Create` to create the instance.
- Wait for it to start up.
### Connect to the Server using ssh
- Open the instance you created on Amazon Lightsail.
- Click on `connect` tab and click on `Account page` at the end of the page then download the Default Private Key.
- Create a folder in the home directory `~` and name it `.ssh`.
- Rename the private key file to `lightsail_key.rsa` and move it into `~/.ssh` folder.
- In the terminal, run `chmod 600 ~/.ssh/lightsail_key.rsa` to change file permission.
- Run `ssh -i ~/.ssh/lightsail_key.rsa ubuntu@YOUR_INSTANCE_PUBLIC_IP` to connect to yor server instance using the terminal.
## Secure your server
### Update all currently installed packages
- Run `sudo apt-get update` update all your repositories for all your apps to all the latest updates lists.
- Run `sudo apt-get upgrade` download and install those packages.
- Run `sudo apt-get dist-upgrade` update all packages to the newest available version. It will also install and remove dependencies as needed.
- Run `sudo shutdown -r now` to shutdown and reboot after shutdown.
### Change the SSH port from 22 to 2200
- Run `sudo nano /etc/ssh/sshd_config`.
- Change the port number from `22` to `2200`.
- Save and exit the file.
- Run `sudo service ssh restart` to restart SSH.
### Firewall configuration of Amazon Lightsail
- Open the instance you created on Amazon Lightsail.
- Click on `Networking` tab, and then click on `Edit rules`.
- Then adding these rules ports 80(TCP), 123(UDP), and 2200(TCP), and delete port 22.
### Configure the Uncomplicated Firewall (UFW)
- Run `sudo ufw status` to display the stauts of the UFW.
- Run `sudo ufw default deny incoming` to deny all incoming connection.
- Run `sudo ufw default allow outgoing` to allow all outgoing connection.
- Run `sudo ufw allow 2200/tcp` to allow all incoming on port 2200 connections.
- Run `sudo ufw allow www` to allow all incoming HTTP port 80 connections.
- Run `sudo ufw allow 123/udp` to allow all incoming on port 123 connections.
- Run `sudo ufw deny 22` to deny all incoming SSH connections.
- Run `sudo ufw enable` to turn UFW on.
- Run `exit`.
- Then run `ssh -i ~/.ssh/lightsail_key.rsa -p 2200 ubuntu@YOUR_INSTANCE_PUBLIC_IP` to connect to yor server instance.

## Give grader access
### Create a new user account named grader
- Run `sudo adduser grader`.
- Then enter a password and info for the new user.
### Give grader the permission to sudo
- Run `sudo visudo` to edits the sudoers file.
- Add the following line after `root ALL=(ALL:ALL) ALL`
```
  grader ALL=(ALL:ALL) ALL
```
- Save and exit the file.
### Create an SSH key pair for grader using the ssh-keygen tool
- Run `ssh-keygen` on the local machine.
- Enter the default path `~/.ssh` and name the key file `grader_key`.
- Enter in a passphrase.
- Run `cat ~/.ssh/grader_key.pub` and copy the contents of the file.
- Log into the grader's virtual machine.
- Run `mkdir .ssh` to create a new dir.
- Run `sudo touch ~/.ssh/authorized_keys` to create a new file.
- Run `sudo nano ~/.ssh/authorized_keys` and paste the content of `grader_key.pub` into this file.
- Run `chmod 700 .ssh` to change the permission of the dir. 
- Run `chmod 644 .ssh/authorized_keys` to change the permission of the file. 
- Run `sudo nano /etc/ssh/sshd_config` file if `PasswordAuthentication` is set to `yes` replace it to `no` to .
- Run `sudo service ssh restart` to restart SSH.
- Run `exit` to back to local machine

## Prepare to deploy the project
- Run `ssh -i ~/.ssh/grader_key -p 2200 grader@YOUR_INSTANCE_PUBLIC_IP` to connect to yor server instance as grader user.
### Configure the local timezone to UTC
- Run `sudo dpkg-reconfigure tzdata`
- Then select `None of the above`
- Then select `UTC`
### Install and configure Apache
- Run `sudo apt-get install apache2` to install apache.
- Run `sudo apt-get install libapache2-mod-wsgi-py3` to install Python 3 mod_wsgi package.
- Run `sudo a2enmod wsgi` to enable mod_wsgi.
- Run `sudo service apache2 restart` to restart apache service.
### Install and configure PostgreSQL
- Run `sudo apt-get install postgresql`
- Run `sudo nano /etc/postgresql/9.1/main/pg_hba.conf` to check that no remote connections are allowed your file should look like this code:
```
local   all             postgres                                peer
local   all             all                                     peer
host    all             all             127.0.0.1/32            md5
host    all             all             ::1/128                 md5
```
- Run `sudo su - postgres` to switch to postgres user.
- Run `psql` to open PostgreSQL terminal.
- Run `CREATE ROLE catalog WITH LOGIN 123 'catalog';` to create the catalog user with a password = 123.
- Run `ALTER ROLE catalog CREATEDB;` to give the catalog user the ability to create databases.
- Run `\q` to exit PostgreSQL terminal.
- Run `exit` to back to grader user.
- Run `sudo adduser catalog` to create a new Linux user called catalog.
- Then enter a password and info for the new user.
- Run `sudo visudo` to edits the sudoers file.
- Add the following line after `root ALL=(ALL:ALL) ALL`
```
  catalog ALL=(ALL:ALL) ALL
```
- Save and exit the file.
- Run `su - catalog` to logged in to catalog user.
- Run `createdb catalog` to create a database named catalog.
- Run `exit` to back to grader user.
### Install git
- Run `sudo apt-get install git` to install git.

## Deploy the Item Catalog project
- Run `mkdir /var/www/catalog`.
- Run `cd /var/www/catalog`.
- Run `sudo git clone https://github.com/abdohfox/Item-Catalog.git catalog` to clone the Item Catalog project.
- Change directory to `/var/www`.
- Run `sudo chown -R grader:grader catalog/` to change the ownershop of the folder.
- Change directory to `/var/www/catalog/catalog.`
- Run `mv cproject.py __init__.py` to rename `cproject.py` to `__init__.py`.
- In `__init__.py` replace 
```
  app.debug = True
  app.run(host='0.0.0.0', port=8000)
```
to 
```
  app.run()
```
- In `__init__.py`, `database_setup.py` and `database_items.py` replace
```
  engine = create_engine('sqlite:///itemcatalog.db')
```
to
```
  engine = create_engine('postgresql://catalog:123@localhost/catalog')
```
- In `__init__.py` replace 
```
  CLIENT_ID = json.loads(
    open('client_secrets.json', 'r').read())['web']['client_id']
```
to
```
  CLIENT_ID = json.loads(
    open(r'/var/www/catalog/catalog/client_secrets.json').read())['web']['client_id']
```

## Install virtual environment and Flask framework
### Install virtual environment
- Change to home directory of grader user.
- Run `sudo apt-get install python3-pip` to install python3.
- Run `sudo apt-get install python-virtualenv` to install the virtual environment.
- Run `cd /var/www/catalog/catalog/`.
- Run `sudo virtualenv -p python3 venv` to create the virtual environment.
- Run `sudo chown -R grader:grader venv/` to change the ownership of the folder to grader.
- Run `. venv/bin/activate` to activate the environment.
- Run this commands to install the required libraries
```
  pip install flask
  pip install sqlalchemy
  pip install psycopg2
  pip install httplib2
  pip install requests
  pip install --upgrade oauth2client
  sudo apt-get install libpq-dev
```
- Run `python3 __init__.py` to test that everything works fine.
- Run `deactivate` to deactivate the virtual environment.
### Setting Up the Virtual Host Configuration
- Run `sudo nano /etc/apache2/mods-enabled/wsgi.conf` and add the following line to it 
```
  WSGIPythonPath /var/www/catalog/catalog/venv/lib/python3.5/site-packages
```
- Run `sudo touch /etc/apache2/sites-available/catalog.conf` to create catalog.conf file and add the following lines to it
```
  <VirtualHost *:80>
    ServerName YOUR_INSTANCE_PUBLIC_IP
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
- Run `sudo a2ensite catalog` to enable the virtual host.
- Run `sudo service apache2 reload` to reload apache service.

### Setting Up Flask application
- Run `sudo touch /var/www/catalog/catalog.wsgi` and add the following lines to it
```
  activate_this = '/var/www/catalog/catalog/venv/bin/activate_this.py'
  with open(activate_this) as file_:
      exec(file_.read(), dict(__file__=activate_this))

  #!/usr/bin/python
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0, "/var/www/catalog/")

  from catalog import app as application
  application.secret_key = "123"
```
- Run `sudo service apache2 restart` to restart apache service.
### Setting Up the Database
- Run `cd /var/www/catalog/catalog/`.
- Run `. venv/bin/activate` to activate the environment.
- Run `python database_setup.py` to setup the database.
- Run `python database_items.py` to fill the database with some data.
- Run `deactivate` to deactivate the virtual environment.
### Disable defualt Apache page
- Run `sudo a2dissite 000-default.conf`.
- Run `sudo service apache2 reload` to reload apache service.

## Setting up OAuth2.0
### Google OAuth
1. Go to [Google APIs Console](https://console.developers.google.com)
2. Sign in to your account or make a new one.
3. Create a New Project.
4. Choose Credentials from the menu on the left.
5. Choose Web application
6. Enter name 'Catalog Item'
7. Set the authorized JavaScript origins = 'http://YOUR_INSTANCE_PUBLIC_IP'.
8. Set the authorized redirect URIs = 'http://YOUR_DOMAIN_NAME'.
9. You will then be able to get the client ID and client secret.
10. Copy the client ID and paste it into the `data-clientid` field in login.html file.
11. On the Google APIs Console page download the JSON file and rename it to client_secrets.json.
12. Place the JSON file into catalog app directory.

### Facebook OAuth
1. Go to [Facebook Developer](https://developers.facebook.com/)
2. Sign in to your account or make a new one.
3. Click on create new app.
4. Enter name 'Catalog Item'
5. Then select 'Integrate Facebook Login' scenario then confirm.
6. Configure the URL site as: http://YOUR_INSTANCE_PUBLIC_IP
7. Create a new file and name it fb_client_secrets.json
8. Then copy this code into it
```
{
  "web": {
    "app_id": "PASTE_YOUR_APP_ID_HERE",
    "app_secret": "PASTE_YOUR_CLIENT_SECRET_HERE"
  }
}
```
9. Copy the App ID and App Secret and paste it into 'fb_client_secrets.json' file
10. Place the JSON file into catalog app directory.

- Run `sudo chown -R www-data:www-data catalog/` to change the owner of all the directories and files.
- Run `sudo service apache2 restart` to restart apache service
- Now you should be able to launch the application at http://YOUR_INSTANCE_PUBLIC_IP
- You can access my app at http://18.130.245.167/

## Resources
- [Set up SSH](https://lightsail.aws.amazon.com/ls/docs/en/articles/lightsail-how-to-set-up-ssh)
- [Connect to the Server using ssh](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstancesLinux.html)
- [Update packages](https://www.ostechnix.com/upgrade-ubuntu-single-command/)
- [Configure SSH](https://www.digitalocean.com/community/tutorials/how-to-use-ssh-to-connect-to-a-remote-server-in-ubuntu)
- [Configure UFW](https://help.ubuntu.com/community/UFW)
- [Add new user and give him permission to sudo](https://www.digitalocean.com/community/tutorials/how-to-edit-the-sudoers-file-on-ubuntu-and-centos)
- [Configure the local timezone to UTC](https://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt)
- [Configure mod_wsgi](https://devops.ionos.com/tutorials/install-and-configure-mod_wsgi-on-ubuntu-1604-1/)
- [Deploy mod_wsgi](http://flask.pocoo.org/docs/1.0/deploying/mod_wsgi/)
- [Secure and configure PostgreSQL](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)
- [Create a new database user has limited permissions](https://www.postgresql.org/docs/9.3/sql-createrole.html)
- [Deploy a Flask Application](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
- [Engine Configuration](https://docs.sqlalchemy.org/en/latest/core/engines.html)
- [Using Python 3 in virtualenv](https://stackoverflow.com/questions/23842713/using-python-3-in-virtualenv)
- [Getting Flask to use Python3](https://stackoverflow.com/questions/30642894/getting-flask-to-use-python3-apache-mod-wsgi)
- [Disable defualt Apache page](https://www.digitalocean.com/community/questions/block-default-apache-page-on-ubuntu-14-04)
- [Permissions for Apache](https://fideloper.com/user-group-permissions-chmod-apache)






















