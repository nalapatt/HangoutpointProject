# HangoutpointProject
# The Problems faced
In this project a ticket booking company is facing problems due to using a waterfall method 
An entertainment company like BookMyShow where users book their tickets have multiple users accessing their web app. Due to less infrastructure availability, they use less machines and provide the required structure. This method includes many weaknesses such as:

- Developers must wait till the complete software development for the test results.
- There is a huge possibility of bugs in the test results.
- Delivery process of the software is slow.
- The quality of software is a concern due to continuous feedback referring to things like coding or architectural issues, build failures, test conditions, and file release uploads.

# The solution
The objective is to implement the automation of the build and release process for
their product.
This was achieved by
- Setting up the Jenkins server in master or slave architecture
- Using the Jenkins plugins to perform the computation part on the Docker containers
- Creating Jenkins pipeline script
- Using the GIT web hook to schedule the job on check-in or poll SCM
- Building an image using the artifacts and deploy them on containers
- Removing the container stack after completing the job

# Steps involved

# Step 1

# Create master and slave EC2 instances in AWS
select Ubuntu 18.04 and default settings and launch instances

# Step 2

# In the master install jenkins and ansible
# Clone Repository that use ansible playbooks to install jenkins and java
 - git clone https://github.com/nalapatt/ansible-role-jenkins.git
 - git clone https://github.com/nalapatt/ansible-role-java.git
 
 # Install python
 - sudo apt install -y git zip python
 
 # Create 2 files for roles and config
git clone https://github.com/nalapatt/HangoutpointProject.git
      
 mv ansible-role-java nalapatt.java
 mv ansible-role-jenkins nalapatt.jenkins
 
 mkdir playbooks
 mkdir playbooks/roles
 mv nalapatt.j* playbooks/roles/
 mv HangoutpointProject/install-jenkins.yaml playbooks/roles/
 cp HangoutpointProject/inv.yaml inv.yaml 
 ls playbooks/roles/
 [which should show] nalapatt.java  nalapatt.jenkins install-jenkins.yml
 
 # Install Ansible
 sudo apt update
 sudo apt install ansible
 # Execute Playbook
 sudo ansible-playbook -i inv.yaml playbooks/roles/install-jenkins.yaml
 
 # Enable Jenkins
 sudo systemctl status jenkins
 sudo systemctl enable jenkins 

# install docker on jenkins and make sure jenkins user has permissions to use docker commands

 
# Install java and maven and configure jenkins with plugins

in a new tab open https://maven.apache.org/download.cgi
Download apache-maven-3.6.3-bin.zip ( if this downloads on your local machine instead of in the VM either download filezilla to transfer the files into the VM

or use SCP or FTP to copy the files from your local to the azure VM)

check is the zip file is dowloaded in the terminal
unzip apache-maven-3.6.3-bin.zip in the terminal
sudo apt-get update
sudo snap install openjdk
sudo apt install openjdk-8-jre-headless

In Jenkins go to Manage Jenkins
Go to Global Tool Configuration
Press Maven installations
Give a Maven name (any name you want)
Give the path ( in my case it is /home/ubuntu/apache-maven-3.6.3/ to check what the path is type pwd)
untick install automatically

Press JDk installations
Give a JDK name ( any name)
Give the path ( in my case it is /usr/lib/jvm/java-11-openjdk-amd64/)(to find path type this, update-alternatives --list java 
(the path is the ones before the jdk file)
untick install automatically
if there is an error saying this is not a jdk diretory, check the path , if correct,
maybe only the jre is installed and not the jdk. so then type this command line sudo apt install default-jdk and then try again

Press Git installations
Give a Git name (I gave git)
Give the path ( in my case it is /usr/bin/git to check what the path is type which git)
untick install automatically
Save

Go to Manage Jenkins
Manage Plugins
Maven Intergration PlugIn in the available column
Click Install without Restart

Now you will be able to see the Maven Build in the Create Job section of Jenkins
Git Plugin in the manage plugins section
Click Install without Restart

# add slave as jenkins user 
sudo useradd -m jenkins
sudo -u jenkins mkdir /home/jenkins/.ssh

# in master
ssh-keygen -t rsa ( just press enter for all the questions) ( it will create an ssh private key is id_rsa and a public key in id_rsa.pub) 
cat .ssh/id_rsa.pub >> .ssh/authorized_keys ( to copy the public keys into the authorized_keys file)
cat .ssh/authorized_keys 

# in slave
sudo -u jenkins vi /home/jenkins/.ssh/authorized_keys (cut and paste and copy the master key here)
esc :wq to save and exit
ssh jenkins@172.31.20.35
see if connection is there and then exit

# in the master copy 
sudo cp ~/.ssh/known_hosts /var/lib/jenkins/.ssh

# in the jenkins UI
manage jenkins
manage nodes and cloud
new node
give a name
click permanent agent
ok
remote root /home/jenkins
label my_slave
use as much as possible
launch agents via ssh
host ip address of slave  (with dots)
add jenkins
ssh user with private key
username jenkins
description jenkins private key credentials
enter directly
cat id_rsa
copy and paste here

# clone the jenkins maven pipeline which has a demo app and test app
https://github.com/nalapatt/JenkinsCI-CDDockerImage.git
# Create a Pipeline
go to the jenkins UI and create a new pipeline project

# Snippet Generator

sample step:Git
url:https://github.com/nalapatt/JenkinsCI-CDDockerImage.git
branch:master
# Add credentials : username and password of github
id:git_credentials
description: git_credentials
Add
select credentials
Generate Pipeline Script
# Scm checkout will contain this snippet
node{
    stage('SCM Checkout'){
        git credentialsId: 'git-creds', git branch: 'dev', url: 'https://github.com/nalapatt/my-app'
    }

# Mvn Clean package
sample step: use a tool from predefined
tool type:Maven
tool:maven-3
generate pipeline script and paste in mvn clean
   stage('Mvn Package'){
        def mvnHome = tool name: 'maven-3', type: 'maven'
        def mvnCMD = "${mvnHome}/bin/mvn"
        sh "$mvnCMD} clean package"
    }

# install docker and make sure you have permissions
# Build Docker Image
stage('Build Docker Image'){
         sh 'docker build -t nalapatt/my-app:2.0.0 .'
    }
    
# Push Docker Image
stage('Build Docker Image'){
         sh 'docker push nalapatt/my-app:2.0.0'
    }  



# Build Triggers
Select Poll SCM
H/15 * * * *

# Pipeline Script from scm
scm - git
url - git where Jenkinsfile resides

# install docker 
install docker on all of them 
sudo apt-get update
sudo apt install docker.io
docker --version

# start docker
sudo service docker start 




