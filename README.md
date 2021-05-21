# HangoutpointProject
In this project a ticket booking company who was facing problems due to using a waterfall method used jenkins to automate their CI/CD pipeline

# Create master and slave EC2 instances in AWS
select Ubuntu 18.04 and default settings and launch instances

# Clone Repository that use ansible playbooks to install jenkins and java

 git clone https://github.com/nalapatt/ansible-role-jenkins.git
 git clone https://github.com/nalapatt/ansible-role-java.git
 
 # Install python
 sudo apt install -y git zip python
 
 # Create 2 files for roles and config
cat inv.yml
[jenkins]
localhost

cat install-jenkins.yml
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

 
 
