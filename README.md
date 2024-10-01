# **Building web application through CI pipeline and utilising static code quality analysis**

This project is to build a  web application through CI pipeline with jenkins, code quality check with sonarqube. 
The builds are triggered using github webhook.

## **Setting up project environment** 

** Initializing EC2 Instance **

Since minimum hardware requirement for sonarqube is quite big, a t2.medium instance is selected instead of t2.micro

Log into AWS console using IAM User account. 
Create an EC2 instance
Select an OS image - Ubuntu
Create a new key pair with any name & download .pem file
Use the same key pair in the EC2 instance creation screen. 
Instance type - t2.medium
Connecting to the instance using ssh

Use the below command to connect to EC2 instance using SSH

```
ssh -i <keypair_name>.pem ubunutu@<IP_ADDRESS>
```

### **Configuring Ubuntu on EC2 instance and installing dependencies**

Update the outdated packages and dependencies. 

```
sudo apt update
```

### **Installing Git**

Install git using the below command. 

```
sudo apt install git
```

### **Installing Jenkins**

```
sudo apt install openjdk-17-jre
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
```

### **Starting Jenkins service**

```
sudo systemctl enable Jenkins
sudo systemctl start Jenkins
```

### **Installing sonarqube**

For sonarqube, a new user is created using adduser command and a password is also set. 
After the new user is created, we need to switch to the newly created user and then run the following commands. 
The username need not be sonarqube but for the project, the username we have used is sonarqube. 

```
apt install unzip
adduser sonarqube
sudo su - sonarqube
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.4.0.54424.zip
unzip *
chmod -R 755 /home/sonarqube/sonarqube-9.4.0.54424
chown -R sonarqube:sonarqube /home/sonarqube/sonarqube-9.4.0.54424
cd sonarqube-9.4.0.54424/bin/linux-x86-64/
./sonar.sh start
```


### **Integrating git and sonarqube with Jenkins for CI**

**Logging into jenkins**

Log into Jenkins using the link
> http://<ec2-public-ip>:8080

As per the instruction use the below command to get the admininstrator password

```
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

Create an admin user after entering the password. 

After login, go to Manage Jenkins > Manage Plugins > Available plugins and install "Sonarqube scanner" plugin

Restart Jenkins after installation of plugin by going to the following link

> http://<ec2-public-ip>:8080/restart

Go to Jenkins Dashboard and create new item. 

Choose "Freestyle Project" , enter a name and click on "OK"

Go to source code management, choose git, provide the details of the git repository and select the main branch. 

> https://github.com/praveendhas-gv/Jenkins-Sonarqube-Docker.git

Go to build trigger section and select github hook trigger for GIT scm polling

For the webhook, go to the settings for the repository "Jenkins-Sonarqube-Docker", select webhooks in the left pane and click on add webhook. 

The payload url should point to the Jenkins server

> http://<ec2-public-ip>:8080/github-webhook/

Content type is Application/json

Click on add webhook to finalize setting up the webhook. 

Any change in code in git scm will now trigger the build job in Jenkins. 

**Logging in sonarqube**

log into sonarqube using the following link 

> http://<ec2-public-ip>:9000

The default username is admin. Password is admin. 

Create a new project manually, enter the project name and click on setup.

Choose Jenkins as CI method to integrate and then choose github as scm. Click on configure analysis and continue to proceed further, and choose other code option for the html project. 

The project key will then be displayed in step 3 which needs to be added to Jenkins. Click on finish next 


Next, go to My Account> Security

Enter a token name, generate token and copy the token. 

Go to jenkins> manage jenkins and click on "Credentials"
Choose system> Global credentials> add credentials and add the sonarqube token as secret text with "sonarqube" as ID

### **Configuring sonarqube in jenkins**

From the Jenkins dashboard> manage jenkins> tools > Click on Add SonarQube scanner
Enter SonarQube as name and choose "Install automatically"

From Jenkins dashboad, click on manage jenkins, click on sytem and scroll down. 
In the sonarqube server section, mention name, sonarqube server url and the authentication token "sonarqube"  which was setup previously


Go go jenkins dashboard, click on the freestype project that was created, choose configure. 

In the build steps section, click on  add build step and choose "Execute sonarqube scanner"
In the analysis properties section, add the project key that was obtained during the step 3 and then click on save and apply. 

Running the jenkins job will now build the application, perform the codequality analysis and display the results in the sonarqube dashboard. 

![sonarqube](https://github.com/user-attachments/assets/b4c23238-638e-40d0-9031-8ded55b5ce59)


