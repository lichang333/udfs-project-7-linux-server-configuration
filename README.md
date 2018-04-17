# Project 7: Linux Server Configuration

## Description

In this project, the student will take a baseline of Linux server installation to host web applications. The task will include secure server from a number of attck vectors, install and vonfigure a database server, and deply a web application in the server.

## Server Login information

IP address: 140.82.47.41

SSH port: 2200

VPS provider: Vultr

## Deploy Log

### 1. Add user "grader" and grant sudo permissions
  * Login the VPS via terminal by using password : ```$ ssh root@140.82.47.41 ```
  * Create a user named "grader" as root: ```# adduser grader```
  * To edit sudoer, type: ```vi /etc/sudoers.d/grader``` or ```visudo```
  * Add this line at the end:

  `grader ALL=(ALL) NOPASSWD: ALL`

  * To Prevent "sudo: unable to resolve host" error:
    - edit: `vi /etc/hosts`
    - add this line to the file:
    ```
    127.0.0.1    vultr
    127.0.1.1    vultr.guest
    ```
### 2. Prevent remote login as root
  * Edit sshd_config: `vi /etc/ssh/sshd_config`
  * On the line "PermitRootLogin", change it to: `PermitRootLogin no`
  * Restart sshd: `service sshd restart`

### 3. Change ssh port from 22 to 2200
  * Edit sshd_config as previous step: `vi /etc/ssh/sshd_config`
  * On the line "Port", change it to: `Port 2200`
  * Restart sshd: `service sshd restart`

### 4. Enable UFW and only allow connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
  * `ufw allow 2200`
  * `ufw allow 80`
  * `ufw allow 123`
  * `ufw enable`
### 5. Generate SSH key and login by using SSH key
  * On the local computer, change a directory for ssh key: `mkdir ssh_key`
  * Generate SSH key on local side: `ssh-keygen -t rsa -f udacity_grader.rsa -C "grader_key" `
  * Lookup the public key: `cat udacity_grader.rsa.pub`
  * Paste the public key on the server side: `vi /home/grader/.ssh/authorized_keys`
  * Login as grader: `ssh -i udacity_grader.rsa grader@140.82.47.41 -p 2200`
### 6. Disable password login  
  * Edit sshd_config:

    `sudo vi /etc/ssh/sshd_config`
  * On the line PasswordAuthentication, change it to "no"

    `PasswordAuthentication no`
  * Restart sshd:

    `service sshd restart`
### 7. Update All system packages
  * list of available packages and their versions：

    `sudo apt-get update`
  * Upgrade all pakages to most recent versions:

    `sudo apt-get upgrade`

### 8. Install unattended-upgrades to keep pakages up-to-date:
  * Install unattended-upgrades:
    `sudo apt install unattended-upgrades apt-listchanges`
  * Edit /etc/apt/apt.conf.d/50unattended-upgrades:
    `vi /etc/apt/apt.conf.d/50unattended-upgrades`
  * Add this line at the end to give root notice:
    ```
    Unattended-Upgrade::Origins-Pattern {
    Unattended-Upgrade::Mail "root";
    };    
    ```
  * Execute with guide program: `sudo dpkg-reconfigure -plow unattended-upgrades`

  * Make sure /etc/apt/apt.conf.d/20auto-upgrades have following stuffs:
    ```
      APT::Periodic::Update-Package-Lists "1";
      APT::Periodic::Unattended-Upgrade "1";
    ```
  * You can also add: `APT::Periodic::Verbose "2";` to the /etc/apt/apt.conf.d/20auto-upgrades

  * check /etc/apt/listchanges.conf, make sure "email_address = root":
    ```
    cat /etc/apt/listchanges.conf
    ```

    The return will be like this:

    ```
    [apt]
    frontend=pager
    which=news
    email_address=root
    email_format=text
    confirm=false
    headers=false
    reverse=false
    save_seen=/var/lib/apt/listchanges.db

    ```

### 9. Local time setup
  * Lookup time zone at first:

    `date -R`
  * Use following command to choose your local timezone:

    `sudo dpkg-reconfigure tzdata`

### 10. Instll fail2ban to prevent Brute-force attack

  * Type following command to install fail2ban: `sudo apt-get install fail2ban`
  * Install sendmail to notice: `sudo apt-get install sendmail`
  * Create jail.local for safe configure:`sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local`
  * Edit jail.local `sudo vi /etc/fail2ban/jail.local`, search for the "destemail" for destination email address
  * To start/stop fail2ban: `sudo service fail2ban stop` or `sudo service fail2ban start`

### 11. Install web dependencies (Apache and mod_wsgi)
  * Install Apache:

    `sudo apt-get install apache2`
  * Install mod_wsgi:

    `sudo apt-get install libapache2-mod-wsgi python-dev`
  * Enable mod_wsgi: `sudo a2enmod wsgi`

  * Start Apache: `sudo service apache2 start`

### 12. Install other web dependencies
  * Install pip: `sudo apt-get install python-pip`
  * Install Flask: `pip install Flask`
  * Install sqlalchemy: `pip install sqlalchemy`
  * Install requests: `pip install requests`
  * Install oauth2client: `pip install oauth2client`
  * ...and other dependencies: `pip install bleach httplib2 python-psycopg2`

### 13. Install virtual environment
  * Install virtualenv: `sudo pip install virtualenv`
  * Create catalog folder: `mkdir /var/www/catalog`
  * Create a virtual environment: `sudo virtualenv venv`
  * Activate the virutal environment `source venv/bin/activate`
  * Change permissions `sudo chmod -R 777 venv`

### 14. Configure the new virtual host
  * Create a virtual host config file: `vi /etc/apache2/sites-available/catalog.conf`
  * Use following code:
  ```
<VirtualHost *:80>
    ServerName 140.82.47.41
    ServerAlias vultr
    ServerAdmin admin@140.82.47.41
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
  * Enable the virtual host: `sudo a2ensite catalog`
  * Reload Apache: `systemctl reload apache2`

### 15 - Deploy PostgreSQL
  * Install libpq-dev and python-dev: ` sudo apt-get install libpq-dev python-dev`
  * Install PostgreSQL: `sudo apt-get install postgresql postgresql-contrib`
  * Switch to user "postgres": `sudo su - postgres`
  * Login PostgreSQL: `psql`
  * Set password for "postgres" : `\password postgres`
  * Create "catalog" for database user and set password:
    `CREATE USER catalog WITH PASSWORD 'password';`
  * Create a database named "catalog" which own by catalog:
    `CREATE DATABASE catalog OWNER catalog;`

  * Give database "catalog" all permissions to catalog:
    `GRANT ALL PRIVILEGES ON DATABASE catalog to catalog;`

  * Exit: `\q`  
  * Exit postgres: `exit`
  * Edit "database_setup.py" and Flask application to :
    ```
    engine = create_engine('postgresql://catalog:database_password@localhost/catalog')
    ```
  * Set up Database:
    ```
    python /var/www/catalog/catalog/database_setup.py
    ```
  * Reload Apache: `sudo service apache2 restart`

  * The web application now should be ready:
    > http://140.82.47.41

### References
  "Changing Default SSH Port in OpenSSH. Https://Www.knownhost.com,  www.knownhost.com/wiki/security/misc/how-can-i-change-my-ssh-port.

  DigitalOcean. Initial Server Setup with Ubuntu 12.04 | DigitalOcean, DigitalOcean, 1 June 2017,  www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-12-04.

  "How to change time and time zone on Ubuntu." Ubuntu修改时区和时间的方法 - CSDN博客,  blog.csdn.net/YEYUANGEN/article/details/52103445.

  DigitalOcean. How To Secure PostgreSQL on an Ubuntu VPS | DigitalOcean, DigitalOcean, 13 Oct. 2016,  www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps.

  “Ubuntu Documentation.” PostgreSQL - Community Help Wiki, help.ubuntu.com/community/PostgreSQL.

  DigitalOcean. How To Install and Use PostgreSQL on Ubuntu 14.04 | DigitalOcean, DigitalOcean, 13 Oct. 2016, www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-14-04.

  DigitalOcean. How To Deploy a Flask Application on an Ubuntu VPS | DigitalOcean, DigitalOcean, 13 Oct. 2016, www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps.

  DigitalOcean. How To Protect SSH with Fail2Ban on Ubuntu 14.04 | DigitalOcean, DigitalOcean, 30 Nov. 2017, www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-ubuntu-14-04.

  Pan, Steven, et al. “Getting started withPostgreSQL.” PostgreSQL新手入门 - 阮一峰的网络日志, www.ruanyifeng.com/blog/2013/12/getting_started_with_postgresql.html.

  DigitalOcean. How to Run Django with mod_wsgi and Apache with a Virtualenv Python Environment on a Debian VPS | DigitalOcean, DigitalOcean, 13 Oct. 2016, www.digitalocean.com/community/tutorials/how-to-run-django-with-mod_wsgi-and-apache-with-a-virtualenv-python-environment-on-a-debian-vps.

  “SSH修改登录端口禁止密码登录并免密登录.” 简书, www.jianshu.com/p/b294e9da09ad.

  “How Do I Disable SSH Login for the Root User?” How Do I Disable SSH Login for the Root User? - Media Temple, mediatemple.net/community/products/dv/204643810/how-do-i-disable-ssh-login-for-the-root-user.

  “Ubuntu Documentation.” UFW - Community Help Wiki, help.ubuntu.com/community/UFW.
