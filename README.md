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
- cat inv.yml
[jenkins]
localhost

- cat install-jenkins.yml
- hosts: jenkins
  connection: local
  vars:
    jenkins_hostname: localhost
    jenkins_admin_username: admin
    jenkins_admin_password: admin
  roles:
    - role: nalapatt.java
      become: yes
    - role: nalapatt.jenkins
      become: yes
      
 mv ansible-role-java nalapatt.java
 mv ansible-role-jenkins nalapatt.jenkins
 
 mkdir playbooks
 mkdir playbooks/roles
 mv nalapatt.j* playbooks/roles/
 mv install-jenkins.yml playbooks/roles/
 ls playbooks/roles/
 [which should show] nalapatt.java  nalapatt.jenkins install-jenkins.yml
 
 # Install Ansible
 sudo apt update
 sudo apt install ansible
 # Execute Playbook
 sudo ansible-playbook -i inv.yml playbooks/roles/install-jenkins.yml
 
 # Enable Jenkins
 sudo systemctl status jenkins
 sudo systemctl enable jenkins 

 
 
