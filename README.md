# kubernetes set up
# both in master and slave
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
sudo swapoff –a
sudo systemctl daemon-reload
sudo systemctl start kubelet
sudo systemctl enable kubelet service

# install docker
sudo apt-get update
sudo apt install docker.io
docker --version

Once installed start on all of them
- sudo service docker start 


# only in master
sudo su -
kubeadm init
exit
now you will see the join token for the worker nodes

mkdir -p $HOME/.kube
sudo cp -i/etc/kubernetes/admin.conf$HOME/.kube/config
sudo chown$(id-u):$HOME/.kube/config
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d'\n')"
kubectl get nodes
kubectl get pods -all-namespaces

# if you lost the token to get it again
kubeadm token create --print-join-command
copy this token and copy in the nodes 
kubectl get nodes
now you should see the master amd nodes
make sure the ports are open to all traffic





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


# 2nd scenario using maven to build , then jenkins and docker to create an image and push into dockerhub and 
# run an ansible playbook to create a container in the slave nodes, also remove the container after it has been deployed


# create 2 ec2 instances, master and slave
create 2 ubuntu 18.04 instances in aws, make sure port 8080 is exposed in the master

 # in slave terminal
 sudo su -
 apt-get update
 hostname jenkins-slave
 logout
 sudo su -
sudo apt install openjdk-8* 
 
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
 
 
 # check if jenkins is running in master
 sudo su -
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
 ssh jenkins@localhost (it will be denied this is just to create a .ssh/known_hosts file)
 ll
 
 cd .ssh/
 ll
 vi authorized_keys
 paste the master public key here
 :wq save and exit
 ip r
 copy the IP address of the slave
 
 # check if ssh from jenkins user to the slaves works in the master
 ssh jenkins@IPaddress of slave ( which was copied from the slave)
 everything should show connection if everything is alright
 exit and go back to your master VM
 





# give docker user permissions in master and server
useradd dockeradmin
passwd dockeradmin
usermod -aG docker dockeradmin

# install ansible in master to deploy container in slave server
sudo apt-get install software-properties-common
sudo apt-add-repository ppa:ansible/ansible
sudo apt-get update
sudo apt-get install ansible
ansible --version
(get the executable location /usr/bin/ansible

sudo su -
vi myhosts.txt
[dev]
ipaddress of slave   ansible_ssh_user=root ansible_ssh_pass=akaadh123
esc :wq
pwd
(this is where the inv file should be )

cd /etc/ansible/ansible.cfg
[defaults]
inventory = /root/myhosts.txt (whereever you created the myhosts.txt file)
ansible_host_key_checking=FALSE
export ANSIBLE_HOST_KEY_CHECKING=False
esc:wq

cd /etc/ansible/hosts
[dev]
slave ansible_ssh_host=ipaddressofslave
esc :wq
ansible dev --list-hosts (this will show the hosts)
ansible dev -m ping (this will show if pingable)

exit

# give ansible user permissions in master and server
useradd ansadmin
passwd ansadmin

visudo 
under read drop in files
add

## Read drop-in files from /etc/sudoers.d ( the # here does not mean a comment)
#includedir /etc/sudoers.d
ec2-user     ALL=(ALL)    NOPASSWD: ALL 
ansadmin     ALL=(ALL)    NOPASSWD: ALL

esc :wq

cd /etc/ssh
vi sshd_config
uncomment passwordauthentication = yes
comment passwordauthentication = no
esc :wq
service sshd restart

# in master
login as ansadmin
ssh-keygen (enter for the questions)(public and private key is generated)

# in slave
ip r ( get ip address )

# in master
cd .ssh
ssh-copy-id ipaddressofslave
create a password that will be used for ansible
you should have connected to the server
exit
sudo vi /etc/ansible/hosts
delete everything
add
[all_hosts]
ipaddressofserver
esc :wq

# jenkins UI
ipaddressofmaster:8080 in browser (example:ec2-52-207-250-4.compute-1.amazonaws.com:8080)
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
copy and paste in browser
continue
update suggested plugins
update password
url suggestions default
save and finish
start using jenkins

# publish over ssh
# ignore for now
manage jenkins
configure system
publish over ssh
hostname
# until here

# install maven through jenkins ui
manage jenkins
global tool configuration
add
mymaven 
check install automatically
apply and save


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
decription docker-hub
variable dockerHubPwd
credentials select docker-hub
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
dev-slave
dev-slave
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







# 3rd scenario 
# 3 VMs ansible jenkins and slave 
create 3 ec2 instances in aws

# name them
ansible
jenkins
slave

# configure jenkins and docker in jenkins server
 
 
# Install docker in jenkins server

sudo apt-get update
sudo apt install docker.io
docker --version

# start docker in jenkins
sudo service docker start 

# install jenkins and jdk in jenkins

sudo apt install openjdk-8* 
sudo wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
    /etc/apt/sources.list.d/jenkins.list'
 sudo apt-get update
 sudo apt-get install jenkins 
 
 # give user permissions 

sudo usermod -a -G docker jenkins
sudo service jenkins restart
sudo chkconfig docker on
sudo service docker start

# configure ansible in ansible ec2 instance

# install ansible in master to deploy container in slave server
sudo apt-get update
sudo apt-get install software-properties-common
sudo apt-add-repository ppa:ansible/ansible
sudo apt-get update
sudo apt-get install ansible
ansible --version
(get the executable location /usr/bin/ansible

# add hosts
sudo vi /etc/ansible/hosts
add
[deployserver]
Public IP address of slave in ec2
[jenkinsserver]
Public IP address of jenkins in ec2
esc :wq

# change the config
/etc/ansible/ansible.cfg

# update jenkins and slave with ansible ssh keys 

# jenkins server update
sudo apt-get update
sudo su -
useradd -d /home/jenkins -m jenkins
passwd jenkins

enter new password
retype password
passwd jenkins passwd -x -1 jenkins (make password so doesnt expire)
su - jenkins
pwd
should show /home/jenkinsserver
mkdir .ssh

# give permissions to the .ssh directory
chmod 700 .ssh/
cd .ssh
ssh-keygen 
enter for all the questions
creates id_rsa and id_rsa.pub ( private and public keys)
cat id_rsa.pub 
vi authorized_keys
copy and paste the id_rsa.pub key here
esc :wq

MAKE SURE the permissions are -rw------- in remotehost /.ssh. 
if not change the permissions by chmod 600
all three files above will be owned by jenkinsserver. 
if not change the owner by below command
chown jenkins:jenkins authorized_keys
chmod 600 authorized_keys
copy this private key to paste into ansible server
cat id_rsa

# create a user in ansible server
sudo su -
useradd -d /home/ansibleuser -m ansibleuser
passwd ansibleuser
enter new password
retype password
passwd ansibleuser passwd -x -1 ansibleuser (make password so doesnt expire)
su - ansibleuser
pwd
should show /home/ansibleuser
exit

cd ../../
chown ansibleuser:ansibleuser /etc/ansible


cd /etc
check the ownership of ansible folder it should be under ansibleuser
cd ansible
vi remotejenkins.key
copy and paste the private key from jenkins server
esc :wq
ll
check the ownership and change it
chmod 600 remotejenkins.key
# check the connection from ansible to jenkins server
ssh -p22 -i remotejenkins.key jenkins@3.86.186.116 (public ec2 ip address)

connect-yes
hope successful

exit
vi /etc/ansible/hosts
[jenkins]
 54.164.166.210 ansible_ssh_user=jenkinsuser ansible_ssh_private_key_file=/etc/ansible/remotejenkins.key 
 esc :wq
 
 ansible jenkins -m ping
 should show success pong 


# configure tomcat in deployment server
 sudo apt-get update
sudo apt install openjdk-8* 
sudo su -
go to the tomcat server 
tomcat 8 
tar.gz right click copy link address ( https://downloads.apache.org/tomcat/tomcat-8/v8.5.66/bin/apache-tomcat-8.5.66.tar.gz)
wget https://downloads.apache.org/tomcat/tomcat-8/v8.5.66/bin/apache-tomcat-8.5.66.tar.gz
tar -xvzf apache-tomcat-8.5.66.tar.gz (the gz file)
mv apache-tomcat-8.5.66  tomcat
cd tomcat
cd bin
./startup.sh
ipaddress:8080
you should see the tomcat server
click manager app
find / -name context.xml
it shows locations of the files with that name 
cd choose the first directory 
create another terminal and connect to the same instance 
cd to the second directory
vi context.xml
comment the lines
valve that entire line
so <!--   -->!
esc :wq

go to the first context.xml in the first connection
vi first directory
and comment out the valve line
<!--  -->!
esc :wq
cd ../
cd conf
vi tomcat-users.xml

go to the role section and add
<role rolename="manager-gui"/>
<user username="tomcat" password="s3cret" roles="manager-gui"/>

<role rolename="manager-script"/>
<user username="deployer" password="deployer" roles="manager-script"/>
esc :wq

manager app 
sign in with tomcat and s3cret
you are signed in

go to the tomcat UI
manager app
should show a sign in 


# add user and ssh keys
sudo apt-get update
sudo su -
useradd -d /home/deployserver -m deployserver
passwd deployserver
enter new password
retype password
passwd deployserver passwd -x -1 deployserver (make password so doesnt expire)
su - deployserver
pwd
should show /home/deployserver
mkdir .ssh

# give permissions to the .ssh directory
chmod 700 .ssh/
cd .ssh
ssh-keygen 
enter for all the questions
creates id_rsa and id_rsa.pub ( private and public keys)
cat id_rsa.pub >> authorized_keys
this will copy the public key to authorized keys


MAKE SURE the permissions are -rw------- in remotehost /.ssh. 
if not change the permissions by chmod 600
all three files above will be owned by deployserver. 
if not change the owner by below command
chown deployserver:deployserver authorized_keys
chmod 600 authorized_keys

cat id_rsa
copy this private key to paste into ansible server


# in ansible server
vi remotedeploy.key
copy and paste the private key from deploy server
esc :wq
ll
check the ownership and change it
chmod 600 remotedeploy.key

# check the connection from ansible to deployment server
ssh -p22 -i remotedeploy.key deployserver@3.86.186.116 (public ec2 ip address)

connect-yes
hope successful
exit

vi /etc/ansible/hosts
[deployuser]
 54.164.166.210 ansible_ssh_user=deployserver ansible_ssh_private_key_file=/etc/ansible/remotedeploy.key 
 esc :wq
 
 ansible deployserver -m ping
 should show success pong 

# change hosts in ansible server
vi hosts
add 
172.31.27.250  ansible_user=root ansible_password=testpass123

sudo vi /etc/ssh/sshd_config

Permitrootlogin delete prohibit password and add yes
passwordauthentication change no to yes
esc :wq

systemctl restart sshd

# jenkins UI
ipaddressofjenkins:8080 in browser (example:ec2-52-207-250-4.compute-1.amazonaws.com:8080)
go to jenkins terminal
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
copy and paste in browser
continue
update suggested plugins
update password
url suggestions default
save and finish
start using jenkins

# manage plugins
publish over ssh
install without restart

# configure system
publish over ssh section
add ssh server
name : ansible
hostname: public ip address of ansible ec2 instance
username: root
advanced
check use password authentication
password passphrase: testpass123
test configuration see if that goes through

# ansible server terminal
cd /etc/ansible/opt
mkdir temp
mkdir playbooks
copy deploy.yml from github
cd temp
mkdir target


# jenkins UI
new item hangout
maven project
source code management
url https://github.com/nalapatt/spring-webapp.git

post build actions
send build artifacts over ssh in drop down
name : ansible
source file: target/*.war
remote directory: //opt/temp

add server
name ansible
execute command:ansible-playbook /opt/playbooks/deploy.yml

save


# 4th scenario
# configure jenkins and docker in jenkins server
 
 
# Install docker in jenkins server

sudo apt-get update
sudo apt install docker.io
docker --version

# start docker in jenkins
sudo service docker start 

# install jenkins and jdk in jenkins

sudo apt install openjdk-8* 
sudo wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
    /etc/apt/sources.list.d/jenkins.list'
 sudo apt-get update
 sudo apt-get install jenkins 
 
 # give user permissions 

sudo usermod -a -G docker jenkins
sudo service jenkins restart
sudo chkconfig docker on
sudo service docker start

# configure ansible in ansible ec2 instance

# install ansible in master to deploy container in slave server
sudo apt-get update
sudo apt-get install software-properties-common
sudo apt-add-repository ppa:ansible/ansible
sudo apt-get update
sudo apt-get install ansible
ansible --version
(get the executable location /usr/bin/ansible

vi /etc/ansible/hosts
[deployuser]
 54.164.166.210 ansible_ssh_user=deployserver ansible_ssh_private_key_file=/etc/ansible/remotedeploy.key 
 esc :wq
 
 ansible deployserver -m ping
 should show success pong 

# change hosts in ansible server
vi hosts
add 
172.31.27.250 

vi /etc/ansible/hosts
[deployuser]
 54.164.166.210 ansible_ssh_user=deployserver ansible_ssh_private_key_file=/etc/ansible/remotedeploy.key 
 esc :wq
 
 ansible deployserver -m ping
 should show success pong 

# change hosts in ansible server
vi hosts
add 
172.31.27.250  ansible_user=root ansible_password=testpass123

sudo vi /etc/ssh/sshd_config
change
#PermitRootLogin prohibit-password
to 
Permitrootlogin yes
change
#passwordauthentication no 
to 
passwordauthentication yes
esc :wq

systemctl restart sshd

# jenkins UIipaddressofjenkins:8080 in browser (example:ec2-52-207-250-4.compute-1.amazonaws.com:8080)
go to jenkins terminal
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
enter
the password that you see below
copy and paste in browser
continue
update suggested plugins
update password
url suggestions default
save and finish
start using jenkins


# manage plugins
make sure ansible plugin is installed
ssh plugin 
ssh agent
ssh build agent
ssh credentials
ssh pipeline
ssh connection
install without restart

# in deploy server set up user
sudo su -
adduser deploy
passwd welcome123 (remember this password)
enter for all and press Y for correct
remember your username and password

# in jenkins master set up config to accept password and hosts file to configure hosts
sudo vi /etc/ansible/hosts/
172.31.26.33 ( private ip address of server that you want to deploy to)

sudo su -
cd /etc/ssh/
vi sshd_config
PermitRootLogin yes
PasswordAuthentication yes
esc :wq
systemctl restart sshd

# in  deploy master set up config to accept password and hosts file

sudo su -
cd /etc/ssh/
vi sshd_config
PermitRootLogin yes
PasswordAuthentication yes
esc :wq
systemctl restart sshd

# configure UI to set up password credentials
# check ssh connection with password

select manage jenkins
configure system
go to add ssh remote hosts
hostname - pvt ip address of deploy node
port 22
add jenkins
credentials
username with password
username - deploy (the deploy server username you set up )
password - welcome123 ( the password that you gave for that user)
id - deploy
description - deploy
add
select deploy in drop down
leave the other boxes blank 
test connection if passed then works 
from the jenkins user

# from jenkins user ssh keygen
sudo su -
adduser jenkins ( already exists)
passwd 
give new passwordwelcome123
su - jenkins

# generate ssh keys
ssh-keygen 
enter for all the questions
cd .ssh
now you will see id_rsa.pub and id_rsa keys 
# copy the public key to the hosts server
ssh-copy-id -i id_rsa.pub deploy@172.31.23.115 ( to copy the files to the deploy server)
press yes
enter password welcome123
key will be added

# check if you can ssh to the host
ssh -i id_rsa deploy@172.31.23.115 ( to see if you can log in to the server)

yeah if you did
now you can check if the authorized_keys exist in the .ssh folder here
exit to go back to the jenkins server VM


# configure jenkins ssh keys credentials

manage jenkins
manage credentials

Stores scoped to Jenkins
go to the global drop down box and add credentials
ssh username with private key
id dev-server
description dev-server
username deploy
private key
enter directly
add
go back to the terminal 
sudo su -
su - jenkins
cd .ssh
cat id_rsa
get the private key copy and paste in the UI
ok

# add credentials for github 
add credentials
username with password
username (github username)
password ( github password)
id github
description github

# add credentials for docker hub
username with secret text
secret ( docker hub password)
id docker-hub
description docker-hub

# new item pipeline
add pipeline script




