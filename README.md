# Project-06-Linux-Server-Configuration

Host Healthy Recipes Web App on Amazon Lightsail.

## Connection Detials

- **IP Address:** http://52.24.142.67
- **Public DNS:** http://ec2-52-24-142-67.us-west-2.compute.amazonaws.com/
- **SSH Port**: 2200

## Steps Made for Hosting the Application:

### 1. Adding user `grader`:

* Create a new user

```
sudo adduser grader
sudo nano /etc/sudoers.d/grader
```
* Then add this line to the file
```
grader ALL=(ALL) NOPASSWD:ALL
```

* Generate Key-Pairs
```
ssh-keygen
```
* After logging as `grader` install the public key:
```
sudo nano .ssh/authorized_keys
sudo chmod 700 .ssh
sudo chmod 664 .ssh
```

* Force Key Based Authentication:
```
sudo nano /etc/ssh/sshd_config
```
and change `passwordAuthenicaition` to `no` then restart `ssh` using:
```
sudo service ssh restart
```
* Change default ssh port from `22` to `2200`:
```
sudo nano /etc/ssh/sshd_config
```
then resatart `ssh` service
```
sudo service ssh force-reload
```

### 2. Starting Firewall:

```
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw allow 2200/tcp
sudo ufw allow www
sudo ufw allow ntp
sudo ufw enable
```

### 3. Intall the Requirements

```
sudo apt-get install python-psycopg2 python-flask
sudo apt-get install python-sqlalchemy python-pip
sudo pip install oauth2client
sudo pip install requests
sudo pip install httplib2
sudo pip install flask-seasurf
```

### 4. Setting up Apache

Run the following command to install Apache
```
sudo apt-get install apache2
```
Then install WSGI
```
sudo apt-get install libapache2-mod-wsgi-py3
```

### 5. Clone the project files
First install `git`:
```
sudo apt-get install git
```
Then create `www` directory:
```
sudo mkdir healthyRecipes
```
Change owner to grader:
```
sudo chown -R grader:grader healthyRecipes
```
Clone the repo:
```
sudo git clone https://github.com/Sara-Kassem/Healthy-Recipes-AWS-Version.git healthyRecipes
```

### 5. Configure Apache to serve the app:

Create a WSGI app:
```
sudo nano /var/www/healthyRecipes/app.wsgi
```

Add the following code to the WSGI file:
```
#!/usr/bin/python3

import sys 

import logging

logging.basicConfig(stream=sys.stderr)

sys.path.insert(0,"/var/www/healthyRecipes/healthyRecipes")

from main import app as application

application.secret_key = 'super_secret_key'
```

Configure `Appache` by typing this command:
```
sudo nano /etc/apache2/sites-enabled/000-default.conf
```
then add the following code after `<VirtualHost *:80>`
```
        ServerName 52.24.142.67 

        ServerAdmin webmaster@localhost

        WSGIScriptAlias / /var/www/healthyRecipes/app.wsgi

        WSGIApplicationGroup %{GLOBAL}

        <Directory /var/www/healthyRecipes/healthyRecipes>
                Order allow,deny
                Allow from all
        </Directory>

        Alias /static /var/www/healthyRecipes/healthyRecipes/static

        <Directory /var/www/healthyRecipes/healthyRecipes/static/>
                Order allow,deny
                                Allow from all
        </Directory>

        ErrorLog ${APACHE_LOG_DIR}/error.log

        LogLevel debug

        CustomLog ${APACHE_LOG_DIR}/access.log combined

```
  
### 6. Set the timezone to UTC
```
sudo timedatectl set-timezone UTC
```
