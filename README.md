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


# 1st scenario dockerize nodejs create an image and deploy on docker hub using jenkins

 after ssh into VM
 then 
 sudo su -
 apt-get update
 # java 1.7 or greater required to run jenkins
 cd /etc/apt/
 apt install openjdk-8* 
 hostname jenkins-master
 logout
 sudo su -
 wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
 sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
    /etc/apt/sources.list.d/jenkins.list'
 apt-get update
 apt-get install jenkins 

 # in slave terminal
 sudo su -
 apt-get update
 hostname jenkins-slave
 logout
 sudo su -
 apt install openjdk-8* -y
 
 # check if jenkins is running
 sudo su
 cd /etc/apt/sources.list.d
 ps faxu | grep jenkins
 su - jenkins
 pwd
 ssh-keygen -t rsa
 enter for all the questions
 note the directory where the file is stored
 cat  /var/lib/jenkins/.ssh/id_rsa.pub (if this is where the public key is stored)
 copy this key and store it somewhere
 
 # in the slave 
 sudo su 
 adduser jenkins
 enter the password and remember what that is
 reenter it 
 just enter for all the other questions
 information correct y
 
 su - jenkins
 ls -lrth
 ls - la
 ssh jenkins@localhost
 ll
 
 cd .ssh/
 ll
 vi authorized_keys
 paste the master public key here
 :wq save and exit
 ip r
 copy the IP address of the slave
 
 # in the master
 ssh jenkins@IPaddress of slave ( which was copied from the slave)
 everything should show connection if everything is alright
 
# go to jenkins UI ipaddress:8080
# in master
logout
cat /var/lib/jenkins/secrets/initialAdminPassword
copy and paste in UI
continue and get plugins

# in jenkins UI
new node
myslave1
myslave1
5
/home/jenkins ( pwd in slave)
myslave1
use node as much as possible
launch agent on execution of command
press the ? from the drop down 
download the agent.jar from the link , right click on the "here" and copy the link address

go to slave terminal 
cd ../ so pwd is /home/jenkins and run wget link address ( something like this wget http://ec2-54-165-96-187.compute-1.amazonaws.com:8080/jnlpJars/agent.jar)
ip r (to get the ip address which is used in the next command )
substitute  the command "ssh hostname java -jar ~/bin/agent.jar" with ssh jenkins@ipaddrofslave java -jar /home/jenkins/agent.jar
keep this agent as much as possible
save 
launch the agent
it should connect

# Install docker
sudo apt-get update
sudo apt install docker.io
docker --version

# start docker
sudo service docker start 

# Build the docker image
git clone https://github.com/nalapatt/DockerizingNodejs.git ( this has all the files needed to dockerize the node js)
cd DockerizingNodejs
docker build -t nalapatt/docker_web_app . ( build an image (your username / with name given in package .json file) in the current directory)
docker images ( see if the image has been created)

# Run the docker image
docker run -p 49160:8080 -d nalapatt/docker-web-app

# go to the browser
localhost:49160
if you are able to see hello world then all is good

# Go to jenkins UI
# enter the docker hub Credentials
Credentials
global credentials
add
docker hub credentials
id dockerhub
description
ok

# add cloudbees docker build plugin
install without restart

# create a pipeline project
# pipeline script
pipeline
script from scm
git
url https://github.com/nalapatt/DockerizingNodejs.git
none
/master
save and build

# Poll scm
Build triggers
Poll scm
H/15 * * * *
( poll scm every 15 mins ,trigger a build when there is a change/commit in code)
save

# change some code in your file
and commit to trigger a build
this will build and push the image into docker hub


# 2nd scenario using maven to build , then jenkins, ansible and docker to create an image and push into dockerhub

# create 2 ec2 instances, master and slave
create 2 ubuntu 18.04 instances in aws, make sure port 8080 is exposed in the master

# Install docker in master
sudo apt-get update
sudo apt install docker.io
docker --version

# start docker in master
sudo service docker start 

# install jenkins and jdk in master

sudo apt install openjdk-8* 
sudo wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
    /etc/apt/sources.list.d/jenkins.list'
 sudo apt-get update
 sudo apt-get install jenkins 
 ipaddressofmaster:8080 in browser (example:ec2-52-207-250-4.compute-1.amazonaws.com:8080)
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
copy and paste in browser
continue
update suggested plugins
update password
url suggestions default
save and finish
start using jenkins

# give user permissions in master
sudo usermod -a -G docker jenkins
sudo service jenkins restart
sudo chkconfig docker on
sudo service docker start
refresh browser
sign in again 

# install maven through jenkins ui
manage jenkins
global tool configuration
add
mymaven 
check install automatically
apply and save

# install ansible in master to deploy container in slave server
sudo apt-get install software-properties-common
sudo apt-add-repository ppa:ansible/ansible
sudo apt-get update
sudo apt-get install ansible
ansible --version
(get the executable location /usr/bin/ansible

# configure ansible in jenkins UI
manage jenkins
manage plugins - available
check ansible
select install without restart 
manage jenkins
global tool configuration
add ansible
myansible
/usr/bin/
save

# create a pipeline project
new item
hangoutproject
pipeline job
ok
select pipeline 
pipeline script

# stage scm checkout
pipeline syntax
snippet generator - git 
url https://github.com/nalapatt/dockeransiblejenkins.git
master
credentials
add jenkins
username with password
github username and password
id github
description github
select this 
generate pipeline script
copy this and paste in pipeline script

# stage maven build
pipeline syntax
declarative directive generator
sample directive - tool
tool type - maven
tool - mymaven 
generate pipeline script
copy and paste in pipeline script

# stage docker build
nalapatt/

# get the version dynamically
pipeline syntax
declarative directive generator
sample directive sh
git rev-parse --short HEAD
check return standard output
generate pipeline script
copy and paste in pipeline script

# declare environmental variables
pipeline syntax
declarative directive generator
sample directive environment
name DOCKER_TAG
value getVersion()
generate pipeline script
copy and paste in pipeline script

# stage docker push

pipeline syntax
declarative directive generator
sample directive: with credentials bind credentials to variables
add secret text
add credentials
secret text
docker hub passowrd
id dockerhub
decription dockerhub
variable dockerHubPwd
credentials select dockerhub
generate pipeline script
copy and paste in pipeline script

save and build and see if everything goes off well

# use ansible playbook to deploy container in server
deploy-docker.yml contains the playbook
change the variable inside the playbook

dev.inv
contains the private ip address (change this)

# stage deploy using ansible playbook 
pipeline syntax
declarative directive generator
sample directive ansible playbook
tool ansible
path deploy-docker.yml 
inv path dev.inv
add ssh credentials jenkins
ssh username with private key
devserver
devserver
ubuntu
add private pem key
copy and paste
add and select this
vault credentials none
disable the host ssh key - check this 
extra parameteres - DOCKER_TAG="${DOCKER_TAG}"
generate pipeline script
copy and paste in pipeline script


save and build
public ip:8080/dockeransible

this should show the text









cd /var/lib/jenkins/tools
this should contain the maven tools






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
 

# Install java and maven and configure jenkins with plugins

in a new tab open https://maven.apache.org/download.cgi
Download apache-maven-3.6.3-bin.zip ( if this downloads on your local machine instead of in the VM either download filezilla to transfer the files into the VM

or use SCP or FTP to copy the files from your local to the azure VM)

check is the zip file is dowloaded in the terminal
unzip apache-maven-3.6.3-bin.zip in the terminal
sudo apt-get update
sudo snap install openjdk
sudo apt install openjdk-8-jre-headless

# In Jenkins UI 
go to Manage Jenkins
Go to Global Tool Configuration
Press Maven installations
Give a Maven name (any name you want)
Give the path ( in my case it is /home/ubuntu/apache-maven-3.6.3/ to check what the path is type pwd)
untick install automatically

Press JDk installations
Give a JDK name ( any name)
Give the path ( in my case it is /usr/lib/jvm/java-11-openjdk-amd64)(to find path type this, update-alternatives --list java 
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

ssh build plugin
ssh agent
install without restart

# in the slave terminal
# give permissions to slave
sudo su 
pwd
mkdir jenkins
chmod 777 jenkins
# install java in slave
sudo apt-get update
sudo snap install openjdk
sudo apt install openjdk-8-jre-headless




# in the jenkins UI


manage jenkins
manage nodes and cloud
new node
give a name my_name_1
click permanent agent
ok
remote root /home/ubuntu/jenkins
label my_slave_1
use as much as possible
launch agents using ssh
host :slave public ip address (the public dns)
add credentials
add jenkins
ssh username with private key
scope global
id jenkins-slave
descr jenkins-slave
username ubuntu
ctrl N create another terminal 
cd downloads 
cat the private pem key
copy and paste into private key add of jenkins ui
click add
select the credentials that was just created
manually trusted key verification
keep agent online as much
save

launch the agent 
it should show agent is connected

restrict where the project can be run as the name of the slave


save 

go to the agent
copy the agent.jar file to the root directory
then execute the commands which are shown in the UI
now agent should show connected
manage jenkins
configure global security
agents - random 
save and apply

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





