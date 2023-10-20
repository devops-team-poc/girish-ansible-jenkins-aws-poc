
# Summary

This project demonstrates integration of AWS, Ansible and Jenkins.

## Project Description


1. A dedicated server for Jenkins
2. A dedicated server for Ansible Control Node
3. Ansible Playbook, which provision and configure 2 EC2 Instances with webserver ( httpd )
4. Add ansible node to Jenkins as a slave node for running Ansible playbooks
5. Configure Jenkins to execute the Ansible Playbook from Ansible Control Node server as part of the CI/CD pipeline to configure new 2 ec2 instances with WebServer.
6. So the Jenkinsfile configuration will do the following:
  
- Get ansible playbooks from Github
- Run ansible playbooks which will provision 2 ec2 instances and then configure HTTPD on them which will show the Public IP of each instances.

## Table of contents

- [Launching two instances on AWS](#launching-two-instances-on-aws)
- [Configure Jenkins on first instance](#configure-jenkins-on-first-instance)
- [Jenkins Setup](#jenkins-setup)
- [Configure Ansible on another instance](#configure-ansible-on-another-instance)
- [Add Ansible node to Jenkins as a slave node](#add-ansible-node-to-jenkins-as-a-slave-node)
- [Git Repository](#git-repository)
- [Creating Pipeline on Jenkins](#creating-pipeline-on-jenkins)
- [Jenkinsfile](#jenkinsfile)
- [Ansible Configuration files](#ansible-configuration-files)
- [Set Environment Variable in Jenkins](#set-environment-variable-in-jenkins)
- [Build the Jenkins job](#build-the-jenkins-job)
- [Thank You](#thank-you)

## Launching two instances on AWS 

The Amazon Web Services (AWS) platform provides more than 200 fully featured services from data centers located all over the world, and is the world's most comprehensive cloud platform.

- Step 1: Create an AWS account/ Login if have already
- Step 2: Open the Amazon EC2 console at https://console.aws.amazon.com/ec2/ 
- Step 3: Click on "Launch instance"
- Step 4: Number of instances = 2
- Step 5: Choose an Amazon Machine Image (AMI) (ex: Ubuntu 22.04)
- Step 6: Choose an Instance Type (ex: t2.medium)
- Step 7: Select key Pair/ Creat new if not available
- Step 8: Configure Network Settings (Select VPC an Security Group)
- Step 9: Configure Storage
- Step 10: Click on "Launch instance"
- Step 11: port 22,80,8080 must be allowed in security group that you have attached with instance.

## Configure Jenkins on first instance
 
Jenkins is an open source continuous integration/continuous delivery and deployment (CI/CD) automation software DevOps tool written in the Java programming language. It is used to implement CI/CD workflows, called pipelines.

Follow the official document:
  https://www.jenkins.io/doc/book/installing/linux/ 

```bash
# installing Java
sudo apt update -y
sudo apt install default-jre
```

```bash
# installing jenkins 
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
```
```bash
# Verify the installation
  sudo jenkins --version
```
```bash
# Start Jenkins 
  sudo systemctl start jenkins
```
```bash
# Enable Jenkins
  sudo systemctl enable jenkins
```
```bash
# Check status of Jenkins
  sudo systemctl status jenkins
```
* Provide jenkins user sudo power:
sudo vim /etc/sudoers.d/jenkins
```bash
jenkins ALL=(ALL) NOPASSWD:ALL
```

## Jenkins Setup

- Step 1: Open any web browser and search "http://public-ip-of-your-jenkins-server:8080"
- Step 2: Unlock Jenkins
	  Enter the password provided by jenkins at "/var/lib/jenkins/secrets/initialAdminPassword"
- Step 3: Customize Jenkins
	  Select "Install suggested plugins"
- Step 4: Create First Admin User
	  Enter details -----> click on "Save and Continue"
- Step 5: Instance Configuration
	  click on "Save and Finish"
- Step 6: Jenkins is ready!
	  click on "Start using Jenkins"

## Configure Ansible on another instance

Ansible is an open source IT automation tool that automates provisioning, configuration management, application deployment, orchestration, and many other manual IT processes.

```bash
# installing ansible
sudo apt update -y
sudo apt install ansible -y
```
```bash
# Verify installation
ansible --version
```
```bash
# installing some packages regarding integration to AWS
sudo apt install python3-pip
pip3 install boto3
pip3 install botocore
```

* Install amazon.aws collection using the ansible-galaxy command:
```bash
ansible-galaxy collection install amazon.aws
```

## Add Ansible node to Jenkins as a slave node

Slave nodes are the "machines" on which build agents run. Jenkins monitors each attached node for disk space, free temp space, free swap, clock time/sync, and response time. A node is taken offline if any of these values go outside the configured threshold.

Follow this video: https://www.youtube.com/watch?v=99DddJiH7lM

Make sure that 'Java' must be installed on Ansible server

* Step 1: Open any web browser and search "http://public-ip-of-your-jenkins-server:8080"
* Step 2: Click on "Manage Jenkins"
* Step 3: Click on "Nodes" under "System Configuration" section
* Step 4: Click on "New Node"
* Step 5: Provide a Node name, select "Permanent Agent", Click on "Create"
* Step 6: Provide all the configurations and click on "Save" 


## Git Repository

We have to create a GitHub Repository, where we will keep following files :

* Jenkinsfile :- This file will be used by Ansible job.
* ansible.cfg :- Ansible configuration file
* Dynamic inventory file :- A dynamic inventory is a script written in Python, PHP, or any other programming language. It comes in handy in cloud environments such as AWS where IP addresses change once a virtual server is stopped and started again.

* Playbook that configure 2 EC2 instances on AWS
* Playbook that configure web server (HTTPD) on both launched instances
* A Jinja2 template file for index.html
* sshkey file :- For Ansible keybased authentication


## Creating Pipeline on Jenkins

- Step 1: Click on "New Item"
- Step 2: Enter an item name and select "Pipeline" projct and click on "OK"
- Step 3: Provide a Description
- Step 4: Select "Pipeline script from SCM" in "Definition" under Pipeline section
- Step 5: Selecti "Git" in SCM section
- Step 6: Provide Repository url where we have all the files
- Step 7: Enter Branch
- Step 8: Enter name of the jenkins file in "Script Path"
- Step 9: Click on "Save"

* When we build the job, the Jenkinsfile will be used by jenkins to run pipeline.


## Jenkinsfile

A Jenkinsfile is a text file that contains the definition of a Jenkins Pipeline and is checked into source control.

```bash
pipeline {
        agent {
                label 'ansible-control'
        }
        stages {
                stage ("Pull Important files" ) {
                                                steps {
                                                        sh "cd /home/ubuntu/ ; git clone https://github.com/GirishAgarwal007/ansible-jenkins-poc.git "
                        }
                }
                stage ("Run the Ansible Playbooks") {
                                                steps {
                                                        sh "cd /home/ubuntu/ansible-jenkins-poc/ ; ansible-playbook ec2_playbook.yml -e key=$key -e region=$region -e insta_type=$insta_type -e ami=$ami -e sg_group=$sg_group -e subnet=$subnet "
                                                        sh " sleep 60"
                                                        sh "cd /home/ubuntu/ansible-jenkins-poc/ ; ansible-playbook web_server.yml "
                        }
                }
        }
}
```

## Ansible Configuration files


* Create an IAM Role on AWS with the permission of "EC2_FULL_ACCESS" and attach it with Ansible Control node

* Sample Dynamic Inventory file
```bash
plugin: aws_ec2
aws_profile: devops
regions:
  - us-east-1

filters:
  tag:Env:
    - Test
    - Prod
  instance-state-name : running

keyed_groups:
  - prefix: ansible
    key: tags

hostnames:
  - ip-address
```

* Ansible main configuration file :- ansible.cfg
```bash
[defaults]
inventory = /path/to/your/dynamic/inventory/file 
private_key_file = /path/to/your/sshkey/file
host_key_checking = False
remote_user = ubuntu 
ask_pass = false

[privilege_escalation]
become = true
become_user = root
become_method = sudo
become_ask_pass = false


[inventory]
enable_plugins = aws_ec2
```

* Playbook that configure 2 EC2 instances on AWS, all the variable used in the playbook will get values from Jenkins Environment Variables.
```bash
- hosts: localhost
  gather_facts: false
  tasks:
  - name: creating key pair
    ec2_key:
       name: "{{ key }}"
       region: "{{ region }}"
       key_material: "{{ lookup('file', '/home/ubuntu/ansible-jenkins-poc/ansible-key') }}"

  - name: Launching first instance
    ec2_instance:
      instance_type: "{{ insta_type }}"
      image_id: "{{ ami }}"
      key_name: "{{ key }}"
      region: "{{ region }}"
      security_group: "{{ sg_group }}"
      vpc_subnet_id: "{{ subnet }}"
      network:
        assign_public_ip: true
      tags:
        Name: web1
        Env: Test
        Team: Dev

  - name: Launching second instance
    ec2_instance:
      instance_type: "{{ insta_type }}"
      image_id: "{{ ami }}"
      key_name: "{{ key }}"
      region: "{{ region }}"
      security_group: "{{ sg_group }}"
      vpc_subnet_id: "{{ subnet }}"
      network:
        assign_public_ip: true
      tags:
        Name: web2
        Env: Prod
        Team: DevOps

``` 
Make sure you have installed "awscli" and configured aws (aws configure) on Ansible Control node.

* Playbook that configure webserver on launched instances
```bash
---
- hosts: ansible_Env_Test, ansible_Env_Prod
  tasks:
      - name: "Installing Apache"
        apt:
           name: apache2
           update_cache: true
           state: latest

      - name: "Start apache Service"
        service:
            name: "apache2"
            state: started
            enabled: true

      - name: "Creating Document Root"
        file:
          path: /home/ubuntu/app/
          state: directory

      - name: "Configure Web Page"
        template:
           src: index.html.j2
           dest: /home/ubuntu/app/index.html

      - name: "Changing apache.conf"
        lineinfile:
           path: /etc/apache2/apache2.conf
           search_string: '<Directory /var/www/>'
           line: <Directory /home/ubuntu/app>

      - name: "Changing sites-available/000-default.conf"
        lineinfile:
           path: /etc/apache2/sites-available/000-default.conf
           search_string: 'DocumentRoot /var/www/html'
           line: DocumentRoot /home/ubuntu/app
        notify:
            - Restart apache service

      - name: "Changing permission for www-data user"
        command: chown -R ubuntu:www-data /home/ubuntu

  handlers:
        - name: "Restart apache service"
          service:
            name: "apache2"
            state: restarted

```

* Jinja2 template file for index.html (index.html.j2)
```bash
<h1> This is an Ansible Managed(Worket) Node </h1>
<h2> Private IP_ADDRESS is : {{ ansible_default_ipv4.address  }} </h2>
<h2> Public IP_ADDRESS is : {{ inventory_hostname }} </h2>
```

## Set Environment Variable in Jenkins

* Step 1: Go to " http://public-ip-of-your-jenkins-server:8080"
* Step 2: Click on "Manage Jenkins"
* Step 3: Click on "system" under "System Configuration" section
* Step 4: under the "Global properties" check the "Environment variables" option
* Step 5: Add variables with "Name" and "Value".
* Step 6: Click on "Save"


## Build the Jenkins job

* Step 1: Go to " http://public-ip-of-your-jenkins-server:8080"
* Step 2: Click on job that we have created
* Step 3: Click on "Build Now" 
* Step 4: Open any browser and search "public-ip-of-any-instance-that-launched-after-building-the-job"

## Thank You

Following this document, we can integrate AWS, Ansible, and Jenkins.

Feel free to adapt this documentation to your specific requirements.

I hope you will find it useful. 

If you have any doubt in any of the step, feel free to contact me. 

THANK YOU

<table>
  <tr>
    <th> <a href="https://www.linkedin.com/in/girish-agarwal-g7" target="_blank"><img src="https://img.icons8.com/color/452/linkedin.png" alt="linkedin" width="30"/><a/>
</th>
  </tr>
</table>
