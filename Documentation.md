# Documentation for Second Deployment 

## Set Up

1. Login to aws console and start an ec2 ubuntu instance with security groups 22, 80 and 8080 with the default VPC given by aws

2. Connect to the ec2 and use the commands below to install jenkins if you don't already have a jenkins server running.
```
sudo apt update && upgrade
sudo apt install default-jre
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt install jenkins
sudo systemctl start jenkins
sudo systemctl status jenkins
```
3. Create another ec2 instance and name it "app-server" in a newly created VPC and public subnet, with security groups 22 and 5000. Run the commands below in the app-server to install all the dependencies
```
sudo apt update && upgrade
sudo apt install default-jre
sudo apt install python3-pip
sudo apt install python3.10-venv
sudo apt install nginx
