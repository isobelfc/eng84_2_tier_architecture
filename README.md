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

## Third Iteration
- create an ec2 instance for db
- install mongodb with required dependencies
- allow access only from the app instance
- connect the app with db to fetch the data
- app to work with reverse proxy without 3000 port, fibonacci, posts

### Process
- create db instance the same as for the app instance above
- however, security group rules should allow SSH from `My IP`, and allow port 27017 from the private IP of the app instance
- SSH into the instance, and run update and upgrade
- to connect to the MongoDB database, run:
```
wget -qO - https://www.mongodb.org/static/pgp/server-3.2.asc | sudo apt-key add -
echo "deb http://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.2.list
sudo apt-get update
sudo apt-get install -y mongodb-org=3.2.20 mongodb-org-server=3.2.20 mongodb-org-shell=3.2.20 mongodb-org-mongos=3.2.20 mongodb-org-tools=3.2.20

sudo mkdir -p /data/db
sudo chown -R mongodb:mongodb /var/lib/mongodb
sudo sed -i 's/127.0.0.1/0.0.0.0/g' /etc/mongod.conf
sudo systemctl enable mongod
sudo service mongod start
```
- connect the app with the db by creating an environment variable
- in the app instance, run `sudo echo "export DB_HOST=mongodb://db_private_ip:27017/posts" >> ~/.bashrc`
- then source the bashrc file with `source ~/.bashrc`
- the two instances are now connected, and you can run `seed.js` to seed the database for post
- then use `node app.js` to run the app and access fibonacci and posts pages

## VPCs
- Virtual Private Cloud
- gives ability to define and control virtual network
- enables you to launch AWS resources into a virtual network that you've defined
- this virtual network closely resembles a traditional network that you'd operate in your data centre
- allows EC2 instances to communicate with eah other, we can also create multiple subnets within our VPC
- benefits us with scalability of infrastructure of AWS

## Internet gateway
- the point which allows us to connect to the Internet
- a gateway that you attach to your VPC to enable communication between resources in your VPC and the internet

## Subnets
- network inside a VPC
- make networking more efficient
- a range of IP addresses in your VPC
- a subnet could have multiple EC2 instances

## Route Tables
- set of rules, called routes
- used to determine where external network traffic is directed

## NACLs
- NACLs are an added layer of defence
- they work at the subnet level
- stateless - you have to have rules to allow the request to come in and to allow the response to go back out

## Security Group
- works as a firewall on the instance level
- they are attached to the VPC and subnet
- they have inbound and outbound traffic rules defined
- stateful - if you allowed inbound rule that will automatically be allowed outbound, even if unspecified

## Ephemeral Ports
- short-lived ports
- automatically allocated based on the demand
- allows outbound responses to clients on the internet
- ports 1024-65535