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
   ``` 
    
   If the configurations are correct, click logs to see "Agent successfully connect and is online"click link to see  👉 https://github.com/jamarikelly/kuralabs_deployment_3/blob/main/Images/Screen%20Shot%202022-10-26%20at%203.26.02%20PM.png
  
### 5.Create a Pipline Build in Jenkins.

   * SSH into the app-server (EC2 in your VPC) and then run the command "nano /etc/nginx/sites-enabled/default"
   * First change the port from 80 to 5000
   * Scroll down, then replace "Location" with 
   "location / { proxy_pass http://127.0.0.1:8000;
                 proxy_set_header Host $host;
                 proxy_set_header X- Forwarded-For $proxy_add_forwarded_for;
                  }"
                   
 look at the link to see the configurations mentioned above 👉 https://github.com/jamarikelly/kuralabs_deployment_3/blob/main/Images/Screen%20Shot%202022-10-24%20at%203.03.51%20PM.png
 
### 5b. Now edit jenkinsfile in repo to scrip shown below:
  ```
  pipeline {
  agent any
   stages {
    stage ('Build') {
      steps {
        sh '''#!/bin/bash
        python3 -m venv test3
        source test3/bin/activate
        pip install pip --upgrade
        pip install -r requirements.txt
        export FLASK_APP=application
        flask run &
        '''
     }
   }
    stage ('test') {
      steps {
        sh '''#!/bin/bash
        source test3/bin/activate
        py.test --verbose --junit-xml test-reports/results.xml
        ''' 
      }
    
      post{
        always {
          junit 'test-reports/results.xml'
        }
       
      }
    }
     
     stage('Test2'){
      steps{
          echo "Testing"
      }
     }
     stage ('Deploy') {
       agent{label 'awsDeploy'}
       steps{
       sh '''#!/bin/bash
       git clone https://github.com/kura-labs-org/kuralabs_deployment_2.git
       cd ./kuralabs_deployment_2
       python3 -m venv test3
       source test3/bin/activate
       pip install -r requirements.txt
       pip install gunicorn
       gunicorn -w 4 application:app -b 0.0.0.0 --daemon
       '''
       }
     }
       
  }
 }
 
 ```
### 6. Now go back to your Jenkins and create a multibranch pipeline and connect it to your github account. Start your build. 

```
i.    Firstly go to github account and select developer settings, select personalize access tokens.
ii.   Select generate new token, name your token then select "repo" and "admin:repo_hook" then generate token. Copy the key.
iii.  Go back to jenkins, select "New Item" then select create "multibranch pipeline", also give your pipeline a name.
iV.   Add a display name and a brief description.
v.    Select source and select github.
vi.   Select add and then select jenkins
vii.  Under username, enter your github name and under password enter the copied token then select add.
viii. Go back to the credentials dropdown menu and select the one just created. 
iix.  Select the url for your repository and paste it, to validate it. if its confirmed, "apply" then "save" and watch the build process.
```
* If the credentials and configurations are correct, below is what you should see;
👇 https://github.com/jamarikelly/kuralabs_deployment_3/blob/main/Images/Screen%20Shot%202022-10-26%20at%202.41.46%20PM.png

* For my build #1 the test and deploy stages failed because dependencies were not installed on my Ec2, please remember to do that. 
* For my build #2 the deploy stage failed because the agent was not set up properly and also again. Download all the dependencies listed at the begining   of the documentation.

### 7. After configuring the issues listed above, the deployment was now successful and another testing stage "Test2" was also added.
  
 *  👉 https://github.com/jamarikelly/kuralabs_deployment_3/blob/main/Images/Screen%20Shot%202022-10-26%20at%202.46.34%20PM.png


### 8. The stack used for this deployment was; linux, python, nginx, and gunicorn.

### 9. The links shown below are for the images for the VPC and also an image for the pipeline used for the deployment.
    
  * for Pipeline 👉 https://github.com/jamarikelly/kuralabs_deployment_3/blob/main/Images/Untitled.jpg
  * for the VPC  👉 https://github.com/jamarikelly/kuralabs_deployment_3/blob/main/Images/Untitled.drawio.png
