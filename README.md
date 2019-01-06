# Linux Server Configuration: Udacity Full Stack Web Development Program

This project helps you understand how to turn a brand-new Linux server into a secure and efficient web application host.

## Task List  

- [x] Take a baseline installation of a Linux server
- [x] Create a new user named grader and grant this user sudo permissions
- [x] Secure your server from a number of attack vectors
- [x] Prepare your server to host your web applications
- [x] Install and configure a database server
- [x] Deploy one of your existing web applications onto it

**Link to the project**:

Public IP address: 54.212.66.31

Accessible SSH port: 2200

Application URL: http://ec2-54-212-66-31.us-west-2.compute.amazonaws.com/

## Installation of a Linux Server

### Starting a New Ubuntu Linux Server Instance on [Amazon Lightsail](https://lightsail.aws.amazon.com)

Follow the steps outlined by the Udacity's *Get started on Lightsail* :

1. Log in to Lightsail. If you don't already have an Amazon Web Services account, you'll be prompted to create one.
2. Once you're logged in, Lightsail will prompt you to create an instance. 
*A **Lightsail instance** is a Linux server running on a virtual machine inside an Amazon datacenter.*
3. Choose an instance image: Ubuntu. There are two settings to make here. First, choose "OS Only" (rather than "Apps + OS"). Second, choose Ubuntu as the operating system.
4. Choose your instance plan: for this project, the lowest tier of instance is just fine.
5. Give your instance a unique  hostname: use any name you like, as long as it doesn't have spaces or unusual characters in it. Click on Create button and wait for your instance to start up. 

###  Connecting to Your Instance with SSH Client

1. Download your private key from your Lightsail Account page and move it into the ~/.ssh directory on your local machine.

```
mv /(current_private_key_address)/LightsailDefaultKey-us-west-2.pem ~/.ssh/
```
2. Change the key file permission so that only owner can read and write

```
  $ chmod 600 ~/.ssh/LightsailDefaultKey-us-west-2.pem
```

2. SSH into server with your public ip and private-public key pair:

```
$ ssh -i LightsailDefaultKey-us-west-2.pem ubuntu@54.212.66.31
```

### Update all available packages and install `finger`

```
$ sudo apt-get update
$ sudo apt-get upgrade
$ apt-get install finger
```

### Configure the local timezone to UTC

Configure timezone using `sudo dpkg-reconfigure tzdata` ( first select `none of the above` and then set timezone to UTC)


## Create a New User (grader) with `sudo` permissions

1. We can **create a new user** by `sudo adduser` command:

```
$ sudo adduser grader
```

2. To **grant the newly created `grader` user `sudo` permissions**, create a new file named `grader` under the `/etc/sudoers.d` directory:

```
$ sudo nano /etc/sudoers.d/grader
```

Copy and paste the following line into `/etc/sudoers.d/grader`: "grader ALL=(ALL:ALL) ALL", then save and close the file (`^O` `Enter` `^X`).  

3. Configure the key-based authentication for grader user:

- Generate an encryption key on your local machine:

```
$ ssh-keygen -f ~/.ssh/graderkey
```
`ssh-keygen` will generate two files: `graderkey` and `graderkey.pub`

- Log into your remote Linux server through ssh as a root user or login as ubuntu user and then switch to root with `sudo -i` command, 
`cd` to `/home/grader` and create  a new `.ssh` directory using `mkdir .ssh`,
then create an `authorized_keys` file under this new /home/grader/.ssh deirectory:

```
$ touch /home/grader/.ssh/authorized_keys.
```

- Copy the content of the grader.pub file from your local machine to the /home/grader/.ssh/authorized_keys file on your remote server. 
- Change permissiosn for .ssh directory and authorized_keys file:

```
$ sudo chmod 700 /home/grader/.ssh
```

```
$ sudo chmod 644 /home/grader/.ssh/authorized_keys
```
- Change the owner of the newly created `.ssh directory` and `authorized_keys` file from root to grader:

```
$ sudo chown grader /home/grader/.ssh
```

```
$ sudo chown grader /home/grader/.ssh/authorized_keys
```

Now you are able to log into your remote server as grader:

```
$ ssh -i graderkey grader@54.212.66.31
```

## Secure your server from a number of attack vectors

### Enforce key-based authentication

Open OpenSSH SSH daemon configuration file (`sshd_config`), find the PasswordAuthentication line and change it to no.

```
$ sudo nano /etc/ssh/sshd_config
$ sudo service ssh restart
```

### Change the SSH port from 22 to 2200

Open OpenSSH SSH daemon configuration file (`sshd_config`), find the `Port 22` line and change in to `Port 2200`
 
```
$ sudo nano /etc/ssh/sshd_config
$ sudo service ssh restart
```
Now you can log into remote VM as a grader or a ubuntu user using the following commands:

```
$ ssh -i graderkey grader@54.212.66.31 -p 2200
```

```
$ ssh -i LightsailDefaultKey-us-west-2.pem ubuntu@54.212.66.31 -p 2200
```


### Disable ssh login for *root* user

Open OpenSSH SSH daemon configuration file (`sshd_config`), find the line `PermitRootLogin` and change it to `no`.

```
$ sudo nano /etc/ssh/sshd_config
$ sudo service ssh restart
```

### Configure the Uncomplicated Firewall (UFW)

Allow incoming connections only for SSH (port 2200), HTTP (port 80), and NTP (port 123):

```
$ sudo ufw default deny incoming
$ sudo ufw default allow outgoing
$ sudo ufw allow 2200/tcp
$ sudo ufw allow www
$ sudo ufw allow ntp
$ sudo ufw enable
```

## Prepare Linux server to host your web applications

###  Install Apache, mod_wsgi

- Get the server responding to HTTP requests. To do this, we use Apache HTTP Server. 
Install Apache using your package manager with the following command: 

```
sudo apt-get install apache2
```

- Configure Apache to hand-off certain requests to an application handler - `mod_wsgi`.
The first step in this process is to install `mod_wsgi`:

```
sudo apt-get install libapache2-mod-wsgi python-dev 
```

Enable mod_wsgi:
```
$ sudo a2enmod wsgi
```

Then configure Apache to handle requests using the WSGI module by editing `/etc/apache2/sites-enabled/000-default.conf` file:

add the following line at the end of the <VirtualHost *:80> block, right before the closing </VirtualHost> line: `WSGIScriptAlias / /var/www/html/myapp.wsgi`

Restart Apache with the `$ sudo service apache2 start` command.


### Install Git and Other Necessary Software

```
$ sudo apt-get install git
$ sudo apt-get install python python-pip
$ sudo pip2 install flask oauth2client Werkzeug
$ sudo pip2 install sqlalchemy flask-sqlalchemy psycopg2 requests
```

## Install and configure a database server

1. Install PostgreSQL:

```
$ sudo apt-get install postgresql
```

2. Change user to postgres:

```
$ sudo su - postgres
```

3. Log into the PostgreSQL console:

```
$ psql
```

4. Create database user catalog and set password:

```
CREATE USER catalog WITH PASSWORD 'password';
```

5. Create database `catalog` with owner `catalog`:

```
CREATE DATABASE catalog OWNER catalog;
```

6. Grant privileges on `catalog` database to user `catalog`:

```
GRANT ALL PRIVILEGES ON DATABASE catalog to catalog;
```

## Deploy your web applications

- Clone your project repository to /var/www/:

```
$ git clone https://github.com/narayaralian/catalog
```

- Make all necessary changes to the `database_setup.py`, `additems.py`, and `project_catalog.py`:

in all three files replace `engine = create_engine('sqlite:///catalog.db')` with 
                           `engine = create_engine('postgresql://catalog:password@localhost:5432/catalog')`


in `project_catalog.py`:

    - move `app.secret_key` out of `if __name__ == '__main__':`
    - replace ` app.run(host='0.0.0.0', port=8000)` with `app.run()`
    - replace ` open('client_secrets.json', 'r').read())['web']['client_id']` with  
          ` open('/var/www/catalog/client_secrets.json', 'r').read())['web']['client_id']`
    - replace `oauth_flow = flow_from_clientsecrets('client_secrets.json', scope='')` with 
          `oauth_flow = flow_from_clientsecrets('/var/www/catalog/client_secrets.json', scope='')`


- In your [Google Developers Console](https://console.developers.google.com/apis/credentials), 
add your project domain (http://ec2-54-212-66-31.us-west-2.compute.amazonaws.com)  to `Authorized JavaScript origins`, 
and update your `client_secrets.json` file.

- In `/var/www/catalog` create a new file names `catalog.wsgi`:

```
#! /usr/bin/env python
import sys
sys.path.insert(0,"/var/www/catalog/")
from project_catalog import app as application
```

- Modify `000-default.conf` in `/etc/apache2/sites-enabled/`:

```
<VirtualHost *:80>
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/catalog
        
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
        
        WSGIScriptAlias / /var/www/catalog/catalog.wsgi
        
         <directory /var/www/catalog>
                WSGIApplicationGroup %{GLOBAL}
                Order deny,allow
                Allow from all
        </directory>
</VirtualHost>
```

- Restart Apache Server:

```
$ sudo apache2ctl restart
```

## Third-party resources

* [Configuring Linux Web Servers](https://www.udacity.com/course/configuring-linux-web-servers--ud299)
* [How To Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps#step-one%E2%80%94-install-and-enable-mod_wsgi)
* [How To Set Up Apache Virtual Hosts on Ubuntu 14.04 LTS](https://www.digitalocean.com/community/tutorials/how-to-set-up-apache-virtual-hosts-on-ubuntu-14-04-lts)
* [How To Add and Delete Users on an Ubuntu 14.04 VPS](https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-an-ubuntu-14-04-vps)
* [Initial Server Setup with Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-14-04)
* [How To Install Git on Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-install-git-on-ubuntu-14-04)
* [Firewall](https://help.ubuntu.com/community/Firewall)
* [How To Secure PostgreSQL on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)

## Authors

* **Nara Yaralyan** - [LinkedIn](https://www.linkedin.com/in/nara-yaralyan-0b35a833/), [GitHub ](https://github.com/narayaralian)


## Acknowledgments

* Special thanks to [TianyangChen](https://github.com/TianyangChen/LinuxServerConfiguration) and [iliketomatoes](https://github.com/iliketomatoes/linux_server_configuration) for really helpful README files.
* This project has been done as part of the Udacity Full-Stack Web Development Program (https://www.udacity.com/course/full-stack-web-developer-nanodegree--nd004). Many thanks to the Udacity team for such a great program!


