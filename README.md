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
  * Paste the public key on the server side: `vi .ssh/authorized_keys`
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
