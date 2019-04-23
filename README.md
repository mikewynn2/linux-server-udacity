# linux-server-udacity

### Project Description

Take a baseline installation of a Linux distribution on a virtual machine and prepare it to host your web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

- IP address: 54.186.238.30

- Accessible SSH port: 2200

### Walkthrough

1. Create new user named grader and give it the permission to sudo
  - SSH into the server through `ssh -i ~/.ssh/udacity_key.rsa root@54.186.238.30`
  - Run `$ sudo adduser grader` to create a new user named grader
  - Create a new file in the sudoers directory with `sudo nano /etc/sudoers.d/grader`
  - Add the following text `grader ALL=(ALL:ALL) ALL`
  - Run `sudo nano /etc/hosts`
  - Prevent the error `sudo: unable to resolve host` by adding this line `127.0.1.1 ip-10-20-52-12`
   
2. Update all currently installed packages
  - Download package lists with `sudo apt update`
  - Fetch new versions of packages with `sudo apt dist-upgrade`

3. Change SSH port from 22 to 2200
  - Run `sudo nano /etc/ssh/sshd_config`
  - Change the port from 22 to 2200
  - Confirm by running `ssh -i ~/.ssh/udacity_key.rsa -p 2200 root@54.186.238.30`
  
4. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
  - `sudo ufw allow 2200/tcp`
  - `sudo ufw allow 80/tcp`
  - `sudo ufw allow 123/udp`
  - `sudo ufw enable`
  
5. Configure the local timezone to UTC
  - Run `sudo dpkg-reconfigure tzdata` and then choose UTC
 
6. As ubuntu user added the key:
  - `sudo su - grader vim .ssh/authorized_keys`

7. Disable ssh login for root user
  - Run `sudo nano /etc/ssh/sshd_config`
  - Change `PermitRootLogin without-password` line to `PermitRootLogin no`
  - Restart ssh with `sudo service ssh restart`
  - Now you are only able to login using `ssh -i ~/.ssh/udacity_key.rsa -p 2200 grader@54.186.238.30`
 
8. Install Apache
  - `sudo apt-get install apache2`

9. Install mod_wsgi
  - Run `sudo apt-get install libapache2-mod-wsgi python-dev`
  - Enable mod_wsgi with `sudo a2enmod wsgi`
  - Start the web server with `sudo service apache2 start`

  
10. Clone the Catalog app from Github
  - Install git using: `sudo apt-get install git`
  - `sudo mkdir circus`
  - Change owner of the newly created catalog folder `sudo chown -R www-data:www-data circus`
  - `cd /circus`
  - Clone your project from github `git clone https://github.com/mikewynn2/circus-item-catalog.git circus`
  - Create a flaskapp.wsgi file, then add this inside:
  ```
  #!/usr/bin/python
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0,"/var/www/circus/circus")

from application import app as application
application.secret_key = '<some secret key here>'
```

11. Install Flask and other dependencies
  - Install pip with `sudo apt-get install python-pip`
  - Install Flask `pip install Flask`
  - Install other project dependencies `sudo pip install httplib2 oauth2client sqlalchemy`

12. Update path of client_secrets.json file
  - `vim application.py`
  - Change client_secrets.json path to `/var/www/circus/circus/client_secrets.json`
  
13. Configure and enable a new virtual host
  - Run this: `sudo vim /etc/apache2/sites-enabled/circus-site.conf`
  - Paste this code:  
  ```
 <VirtualHost *:80>
                ServerName mywebsite.com
                ServerAdmin admin@mywebsite.com
                WSGIScriptAlias / /var/www/circus/flaskapp.wsgi
                <Directory /var/www/circus/circus/>
                        Order allow,deny
                        Allow from all
                </Directory>
                Alias /static /var/www/circus/circus/static
                <Directory /var/www/circus/circus/static/>
                        Order allow,deny
                        Allow from all
                </Directory>
                ErrorLog ${APACHE_LOG_DIR}/error.log
                LogLevel warn
                CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
  ```
  - Enable the virtual host `sudo a2ensite circus-site`

14. Restart Apache 
  - `sudo service apache2 restart`
  
15. Visit site at [http://54.186.238.30](http://54.186.238.30)


## References:
https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps

https://www.digitalocean.com/community/tutorials/how-to-tune-your-ssh-daemon-configuration-on-a-linux-vps

https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu-16-04
