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

  
