# Documentation for Third Deployment 

## Set Up

### 1. Login to aws console and start an ec2 ubuntu instance with security groups 22, 80 and 8080 with the default VPC given by aws

### 2. Connect to the ec2 and use the commands below to install jenkins if you don't already have a jenkins server running.
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
after jenkins is installed and started, login by filling out username and etc.

### 3. Create another ec2 instance and name it "app-server" in a newly created VPC and public subnet, with security groups 22 and 5000. Run the commands below in the app-server to install all the dependencies
```
sudo apt update && upgrade
sudo apt install default-jre
sudo apt install python3-pip
sudo apt install python3.10-venv
sudo apt install nginx
```
### 4. The other step is to configure and connect a jenkins agent.

* Enter the jenkins server and select the Build Executor Status
* Select the "+ New Node" to configure and add the agent. Enter the node name "awsDeploy" and select "Permanent Agent" and then create 
* After that, you'll be prompted, the configurations should be similar as below: 
 ```
  Name: awsDeploy
  Description: Deployment server
  Number of executors: 1
  Remote root directory: /home/ubuntu/agent 
  Labels: awsDeploy
  Usage: only build jobs with label
  Launch method: launch agent via ssh
  Host: "IP of your EC2"
  Host key verification strategy : non verifying verification strategy
  Availability: keep this agent online as much as possible 
  Credential steps {
                     * Select "Add" => "Jenkins" => Kind: "SSH username with private key"
                     * Enter the ID, Description, username
                     * To add the key, select "Enter Directly" => select "add" => paste the private key into the white box and save.
                     }
                     
    To look at configurations click link 👉
  ```
  
### 5.Create a Pipline Build in Jenkins.

   * SSH into the app-server (EC2 in your VPC) and then run the command "nano /etc/nginx/sites-enabled/default"
   * First change the port from 80 to 5000
   * Scroll down, then replace "Location" with 
   "location / { proxy_pass http://127.0.0.1:8000;
                 proxy_set_header Host $host;
                 proxy_set_header X- Forwarded-For $proxy_add_forwarded_for;
                  }"
                   
 ![Alt text] (https://github.com/jamarikelly/kuralabs_deployment_3/blob/main/Images/Screen%20Shot%202022-10-26%20at%203.26.02%20PM.png? raw=true "Title")
                   
### 5b. Now edit jenkinsfile in repo to scrip shown below:
  
