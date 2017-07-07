# linux-server

## User Info
* Public IP address: 54.160.215.248
* SSH port: 2200
* URL to my hosted web application: 54.160.215.248

## Update and upgrade all currently installed packages
* sudo apt-get update
* sudo apt-get upgrade
* sudo apt-get autoremove

## Create new user grader
* sudo adduser grader
* Enter password and other information

## Give grader user sudo access
* Copy a new file for grader: sudo cp /etc/sudoers.d/ubuntu /etc/sudoers.d/grader
* Edit this file to give user access: sudo nano /etc/sudoers.d/grader
* Change "root" to "grader" and save

## Generate key-pair and disable password-base login:
* On local terminal run command: ssh-keygen and specify location for key-pair
* On server: Make a new directory: mkdir .ssh
* touch .ssh/authorized_keys
* Copy and paste public key to authorized_keys
* Run command to set access permission: chmod 700 .ssh
* Run command: chmod 644 .ssh/authorized_keys

## Change port from 22 to 2200
* sudo ufw default deny incoming
* sudo ufw default allow outgoing
* Allow incoming TCP packets on port 2200 (SSH): sudo ufw allow 2200/tcp
* Allow incoming TCP packets on port 80 (HTTP): sudo ufw allow www
* Allow incoming UDP packets on port 123 (NTP): sudo ufw allow ntp

## Configure the local timezone to UTC:
* Run command date to see what is your timezone, mine is UTC as default

## Install and configure Apache to serve a Python mod_wsgi application
* Install Apache web server:
$ sudo apt-get install apache2
summary of software you installed and configuration changes made
SSH key you created for the grader user
