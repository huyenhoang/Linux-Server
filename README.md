# Linux-Server Configuration
Linux Config for deployment of a Flask App.

Public IP: 54.191.164.45
Host name: http://ec2-54-191-164-45.us-west-2.compute.amazonaws.com/

## Setting up Lightsail & SSH into root user
1. Set up Lightsail account and instance at https://lightsail.aws.amazon.com/. 
  * Choose the Ubuntu for instnce image.
2. Download the Instance’s Lightsail Private Key.
3. Move it to your ~/.ssh directory (I renamed the file to Lightsail.pem to keep it simple). You can do this by placing the file on your desktop and then open up the terminal and enter:
  * `cd desktop`
  * `mv ~/desktop/Lightsail.pem  ~/.ssh/`
4. Set the file permission to 600 for the Lightsail Private Key.
  * `chmod 600 ~/.ssh/Lightsail.pem`
5. SSH into the instance as root user (from local terminal).
  * `ssh -i ~/.ssh/Lightsail.pem ubuntu@54.191.164.45`

## Create new user grader & give sudo access
6. Once logged in, create a new user named `grader` on the remote machine.
  * `sudo adduser grader`
  Optional: You can check if this user is added with finger.
  * `sudo apt-get install finger`
  * `finger grader`
7. Give grader access to sudo.
  * `sudo nano /etc/sudoers.d/grader`
  * Add this text into the file: `grader ALL=(ALL:ALL) ALL`
Or alternatively you can do this,
  * `sudo visudo`
  * Then add the following line below `root ALL=(ALL:ALL) ALL`
  * `grader ALL=(ALL:ALL) ALL`

## Disable Root login & force key-based authentication
8. Update Port to 2200.
  * `sudo nano /etc/ssh/sshd_config`
  * Change port from `22` to `2220`
9. Disable root login.
  * Change `Permit RootLogin` to `no`
10. Enforce Key-based authentication
  * Change or keep `PasswordAuthentication` to `yes` (won't update this to `no` until a later step).
  * add `AllowUsers grader` at end of the file to allow login with grader.
  * `sudo service ssh restart`
11. Update Lightsail instance.
  * Add 2200 as inbound custom TCP Rule port
12. Login as grader from local machine.
  * `ssh -v grader@"54.191.164.45" -p 2200`
  * Enter password created at Step 6 (`PasswordAuthentication` is still `yes` from Step 10)
  
  ## Key Pair Authentication
13. Generate key-pair on local machine.
  * `ssh-keygen`
  * Name it `/Users/username/.ssh/grader` (username & grader will differ from user to user)
14. Read & copy the public key
  * `cat ~/.ssh/grader.pub`
  * Copy public key.
15. On the virtual machine, make new directory called .ssh within the home directory of the grader.
  * `mkdir .ssh`
16. Make authorized key file
  * Touch `.ssh/authorized_keys`
17. Open up that authorized keys file and past public key into it.
  * `sudo nano .ssh/authorized_keys`
  * Save it
18. Change permission of .ssh file
  * `sudo chmod 700 /home/grader/.ssh`
  * `sudo chmod 644 /home/grader/.ssh/authorized_keys`
19. Now make it so you cannot log in with password.
 * `sudo nano /etc/ssh/sshd_config`
 * change `password` to `no`
20. If everything works, you can login from local machine as `grader`.
 * `sudo grader@54.191.164.45 -p 2200 -i ~/.ssh/grader`
 
 Resource: [Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-ubuntu-1604), [Digital Ocean](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-16-04)
 
 
 ## Update all currently installed packages
21. To update packages:
  * `sudo apt-get update`
22. To ugprade packages: 
  * `sudo apt-get upgrade`

## Configure Time Zone
23. Set to UTC
  * `sudo dpkg-reconfigure tzdata`
  * Select `none of the above`, then `UTC`

Resource: [askubuntu](https://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt)

## Configure Firewall
24. Check firewall status.
  * sudo ufw status
25. Block incoming connections on all the ports.
  * `sudo ufw default deny incoming`
26. Allow outgoing connections on all ports.
  * `sudo ufw default allow outgoing`
27. Allow incoming connection for SSH(port 2200)
 * `sudo ufw allow 2200/tcp`
28. Allow incoming connection for HTTP(port 80)
  * `sudo ufw allow 80/tcp`
29. Allow incoming connection for NTP(port 123)
  * `sudo ufw allow 123/udp`
30 Check added rules.
  * `sudo ufw show added`
31. Enable the firewall
  * `sudo ufw enable`
32. Check if firewall is enabled.
  * `sudo ufw status`

## Setting up Apache Server
33. Install apache
  * `sudo apt-get install apache2`
34. Install mod_wsgi
  * `sudo apt-get install libapache2-mod-wsgi python-dev`
35. Enable mod_wsgi andstart the server
  * `sudo a2enmod wsgi`
  * `sudo service apache2 start`
  
## Install version control and clone catalog repository
36. Install git
  * sudo apt-get install git`
37. Clone the catalog app from Github
  * `cd /var/www`
  * `sudo mkdir catalog`
  * Change owner of the newly create `catalog` folder: `sudo chown -R grader:grader catalog`
  * `cd catalog`
  * `git clone https://github.com/huyenhoang/catalog-app catalog`
38. Change the cloned `application.py` file to `__init__.py` using the `mv` command
  * `mv application.py __init__.py`
39. To make git repository inaccessible via the web browser with `.htacess` file inside `/var/www/catalog` (the root of the server):
  * `touch .htaccess`
  * `nano .htaccess`
  * Add `RedirectMatch 404 /\.git` and save the file.

Resource: [Stackoverflow](https://stackoverflow.com/questions/6142437/make-git-directory-web-inaccessible)

## Python and other dependencies
40. Install Python framework pip
  * `sudo apt-get install python-pip` (within `/var/www/catalog`)
  * `pip install-upgrade pip`. 
  Note: with pip 10.0.0, there is a bug so in linux, you need to modify file `/usr/bin/pip`:
  * `sudo nano /usr/bin/pip`
  * change the content to this:
  
    `from pip import __main__`
    
    `if __name__ == '__main__':`
    
    `sys.exit(__main__._main())`

 Resource: [Stackoverflow](https://stackoverflow.com/questions/28210269/importerror-cannot-import-name-main-when-running-pip-version-command-in-windo)
 
41. Install all other dependencies for catalog app.
  * `sudo pip install httplib2`
  * `sudo pip install oauth2client`
  * `sudo pip install sqlalchemy`
  * `sudo pip install psycopg2`
  * `sudo pip install requests`
  * `sudo pip install Flask-SQLAlchemy`
  * `sudo pip install flask-seasurf` - this is optional, but is a good idea if you want to prevent cross-site request forgery (CSRF) while using Flask.

## Configure virtual host
42. Install virtual environment (stay in `/var/www/catalog`).
  * Install virtual environment: `sudo pip install virtual env` (within `/var/www/catalog`)
  * Create a new virtual environment: `sudo virtualenv venv`
  * Activate virtual environment: `source venv/bin/activate`
  * Modify the permissions for the virtual environment: `sudo chmod -R 777 venv`
43. Install Flask.
  * `sudo pip install Flask`
44. Run the application.
  * `sudo python __init__.py`
  * Control C to quit, then `deactivate`
  
Resource: [Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps),
[Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-run-django-with-mod_wsgi-and-apache-with-a-virtualenv-python-environment-on-a-debian-vps)

45. Configure & enable new Virtual Host Config file.
  * `sudo nano /etc/apache2/sites-available/ctalog.conf`
  * Add the below codes:
    `<VirtualHost *:80>`     
    `ServerName 54.191.164.45`     
    `ServerAdmin grader@54.191.164.45`
    `ServerAlias    ec2-54-191-164-45.us-west-2.compute.amazonaws.com`
    `WSGIScriptAlias / /var/www/catalog/catalog.wsgi `    
    `<Directory /var/www/catalog/catalog/>`         
    `Order allow,deny`         
    `Allow from all`      
    `</Directory>`     
    `Alias /static /var/www/catalog/catalog/static`     
    `<Directory /var/www/catalog/catalog/static/>`         
    `Order allow,deny`         
    `Allow from all`     
    `</Directory>`     
    `ErrorLog ${APACHE_LOG_DIR}/error.log`     
    `LogLevel warn`     
    `CustomLog ${APACHE_LOG_DIR}/access.log combined` 
    `</VirtualHost>`



## Installing & Securing PosgreSQL DB
Resource: [Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-16-04)
[Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)
