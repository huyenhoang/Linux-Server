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
 
 
