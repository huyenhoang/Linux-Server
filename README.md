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
