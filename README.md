# 2 Tier Architecture with AWS

## 2 Tier App Deployment on AWS
- Launch an ec2 instance with correct version of ubuntu
- ssh into the instance
- update 
- upgrade
- install nginx 
- access nginx page with public IP
- share the IP in the chat

### Process
- create a new instance with Ubuntu 16.04
- `t2 micro` is big enough for this task, if needed we could scale up to a bigger server
- for this iteration use the default subnet
- make sure to name appropriately
- create a new security group
- allow SSH from `My IP` only
- allow HTTP from anywhere
  
### Security Group vs NACL
- a security group acts as a firewall only on the instance level
- NACL works as a firewall on the subnet level

### Once instance is created
- to ssh in, use copy the command from `Connect to instance`
- e.g. `ssh -i "DevOpsStudent.pem" ubuntu@ec2-ip.eu-west-1.compute.amazonaws.com`
- `sudo apt-get update -y` to update
- `sudo apt-get upgrade -y` to upgrade
- `sudo apt-get install nginx -y` to install nginx
- `sudo systemctl status nginx` to check nginx status
- go to the public IP address to check the nginx page is visible

## 2nd Iteration
- copy code from OS to AWS EC2 app with scp command
- install required dependencies for nodejs
- launch the nodeapp with public IP

### Process
- use the `scp` (secure copy) command to copy folders and files
- `scp -i ~/.ssh/DevOpsStudent.pem -r ~/dev_env ubuntu@ip:~/dev_env`
- this takes a while to run
- navigate to the folder containing the `provision.sh` file
- make the file executable with `chmod +x provision.sh`
- convert dos2unix
```
wget "http://ftp.de.debian.org/debian/pool/main/d/dos2unix/dos2unix_6.0.4-1_amd64.deb"

sudo dpkg -i dos2unix_6.0.4-1_amd64.deb
```
- `dos2unix provision.sh`
- run provision file with `./provision.sh`
- allow all traffic from port 3000 in security group inbound rules
- can now run `app.js` and access app pages on port 3000

### Reverse Proxy
- add in a reverse proxy with
```
sudo echo "server {
    listen 80;

    server_name _;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade \$http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host \$host;
        proxy_cache_bypass \$http_upgrade;
    }
}" | sudo tee /etc/nginx/sites-available/default
```
- restart nginx with `sudo systemctl restart nginx`
- the app pages can now be accessed without including the port