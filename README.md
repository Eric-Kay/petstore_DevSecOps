# Ansible – DevSecOps Petshop project | Jenkins CI/CD
![3tier](https://github.com/Eric-Kay/petstore_DevSecOps/assets/126447235/5a768369-a26f-4b12-adb0-32aed0e84f35)

Hello friends, we will be deploying a Petshop Java Based Application. This is an everyday use case scenario used by several organizations. We will be using Jenkins as a CICD tool and deploying our application on a Docker container and Kubernetes cluster. Hope this detailed blog is useful.

We will be deploying our application in two ways, one using Docker Container and the other using K8S cluster.

STEPS:

Step 1 -- Create an Ubuntu(22.04) T2 Large Instance

Step 2 -- Install Jenkins, Docker and Trivy

Step 3 -- Install Plugins like JDK, Sonarqube Scanner, Maven, OWASP Dependency Check

Step 4 -- Configure Sonar Server in Manage Jenkins

Step 5 -- Install OWASP Dependency Check Plugins

Step 6 -- Docker plugin and credential Setup

Step 7 -- Adding Ansible Repository in Ubuntu and install Ansible

Step 8 -- Kuberenetes Setup

Step 9 -- Master-slave Setup for Ansible and Kubernetes

## __STEP1:__ Create an Ubuntu(22.04) T2 Large Instance
Launch an AWS T2 Large Instance. Use the image as Ubuntu. You can create a new key pair or use an existing one. Enable HTTP and HTTPS settings in the Security Group and open all ports (not best case to open all ports but just for learning purposes it’s okay).

![Screenshot 2024-03-31 225000](https://github.com/Eric-Kay/petstore_DevSecOps/assets/126447235/7c51f123-bdc1-4ad8-ab88-57e236b85356)

## __STEP2:__  Install Jenkins, Docker and Trivy

2A — To Install Jenkins
2A — To Install Jenkins

Connect to your console, and enter these commands to Install Jenkins
```bash
vi jenkins.sh #make sure run in Root (or) add at userdata while ec2 launch
```
```bash
#!/bin/bash
sudo apt update -y
#sudo apt upgrade -y
wget -O - https://packages.adoptium.net/artifactory/api/gpg/key/public | tee /etc/apt/keyrings/adoptium.asc
echo "deb [signed-by=/etc/apt/keyrings/adoptium.asc] https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | tee /etc/apt/sources.list.d/adoptium.list
sudo apt update -y
sudo apt install temurin-17-jdk -y
/usr/bin/java --version
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
                  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
                  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
                              /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update -y
sudo apt-get install jenkins -y
sudo systemctl start jenkins
sudo systemctl status jenkins
```

```bash
sudo chmod 777 jenkins.sh
./jenkins.sh    # this will installl jenkins
```

Once Jenkins is installed, you will need to go to your AWS EC2 Security Group and open Inbound Port 8080 and 8090, 9000 for sonar, since Jenkins works on Port 8080.

But for this Application case, we are running Jenkins on another port. so change the port to 8090 using the below commands.

```bash
sudo systemctl stop jenkins
sudo systemctl status jenkins
cd /etc/default
sudo vi jenkins   #chnage port HTTP_PORT=8090 and save and exit
cd /lib/systemd/system
sudo vi jenkins.service  #change Environments="Jenkins_port=8090" save and exit
sudo systemctl daemon-reload
sudo systemctl restart jenkins
sudo systemctl status jenkins

```

Now, grab your Public IP Address

```bash
<EC2 Public IP Address:8090>
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

![Screenshot 2024-03-18 232236](https://github.com/Eric-Kay/netflix-clone-on-kubernetes/assets/126447235/37441fee-47da-43b3-8591-0c86eb096624)

Unlock Jenkins using an administrative password and install the suggested plugins.

2B — Install Docker

```bash
sudo apt-get update
sudo apt-get install docker.io -y
sudo usermod -aG docker $USER   #my case is ubuntu
newgrp docker
sudo chmod 777 /var/run/docker.sock
```
After the docker installation, we create a sonarqube container (Remember to add 9000 ports in the security group).
```bash
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
```
![Screenshot 2024-03-18 233551](https://github.com/Eric-Kay/netflix-clone-on-kubernetes/assets/126447235/e0f4ec44-8e4e-44b0-a22b-6fd7e841ce3b)

Enter username and password, click on login and change password

```bash
username admin
password admin
```

2C — Install Trivy

```bash
vi trivy.sh
```

```bash
sudo apt-get install wget apt-transport-https gnupg lsb-release -y
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null
echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy -y

```
Next, we will log in to Jenkins and start to configure our Pipeline in Jenkins

## __STEP3:__  Install Plugins like JDK, Sonarqube Scanner, Maven, OWASP Dependency Check
3A — Install Plugin
Goto Manage Jenkins →Plugins → Available Plugins →

Install below plugins

1 → Eclipse Temurin Installer (Install without restart)

2 → SonarQube Scanner (Install without restart)

3B — Configure Java and Maven in Global Tool Configuration
Goto Manage Jenkins → Tools → Install JDK(17) and Maven3(3.6.0) → Click on Apply and Save

3C — Create a Job
Label it as PETSHOP, click on Pipeline and OK.
![Screenshot 2024-03-31 233442](https://github.com/Eric-Kay/petstore_DevSecOps/assets/126447235/a5b5356a-b675-4a25-9949-804167fe413e)

Enter this in Pipeline Script,

```bash
pipeline{
    agent any
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    stages{
        stage ('clean Workspace'){
            steps{
                cleanWs()
            }
        }
        stage ('checkout scm') {
            steps {
                git branch: 'main', url: 'https://github.com/Eric-Kay/petstore_DevSecOps.git'
            }
        }
        stage ('maven compile') {
            steps {
                sh 'mvn clean compile'
            }
        }
        stage ('maven Test') {
            steps {
                sh 'mvn test'
            }
        }
   }
}
```

## __STEP4:__  Configure Sonar Server in Manage Jenkins

Grab the Public IP Address of your EC2 Instance, Sonarqube works on Port 9000, so <Public IP>:9000. Goto your Sonarqube Server. Click on Administration → Security → Users → Click on Tokens and Update Token → Give it a name → and click on Generate Token

+ Goto Jenkins Dashboard → Manage Jenkins → Credentials → Add Secret Text.
+ Now, go to Dashboard → Manage Jenkins → System and configure system.
+ Goto Administration–> Configuration–>Webhooks and insert the jenkins URL.

![Screenshot 2024-04-01 141650](https://github.com/Eric-Kay/petstore_DevSecOps/assets/126447235/2dee5554-a686-45f4-8520-70ceebb68bc6)

Add details.

```bash
#in url section of quality gate
<http://jenkins-public-ip:8090>/sonarqube-webhook/
```
Go to our Pipeline and add Sonarqube Stage in our Pipeline Script.

```bash
#under tools section add this environment
environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
# in stages add this
stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Petshop \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=Petshop '''
                }
            }
        }
        stage("quality gate"){
            steps {
                script {
                  waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
                }
           }
        }
```
## __STEP5:__  Install OWASP Dependency Check Plugins.

+ Goto Dashboard → Manage Jenkins → Plugins → OWASP Dependency-Check. Click on it and install it without restart.
+ Goto to manage Jenkins → tools → Dependency-Check to configure OWASP Dependency.

Now go configure → Pipeline and add this stage to your pipeline and build.

```bash
stage ('Build war file'){
            steps{
                sh 'mvn clean install -DskipTests=true'
            }
        }
        stage("OWASP Dependency Check"){
            steps{
                dependencyCheck additionalArguments: '--scan ./ --format XML ', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
```


## __STEP6:__ Docker plugin and credential Setup

We need to install the Docker tool in our system, Goto Dashboard → Manage Plugins → Available plugins → Search for Docker and install these plugins

Docker

Docker Commons

Docker Pipeline

Docker API

docker-build-step

and click on install without restart.

+ Now, goto Dashboard → Manage Jenkins → Tools → Add DockerHub Username and Password under Global Credentials.
+ Create a personal Access token from the docker hub which is used for ansible-playbook.

## __STEP7:__ Adding Ansible Repository in Ubuntu

Step1:Update your system packages:

```bash
sudo apt-get update
```
Step 2: First Install Required packages to install Ansible.

```bash
sudo apt install software-properties-common
```
Step3: Add the ansible repository via PPA

```bash
sudo add-apt-repository --yes --update ppa:ansible/ansible
```
Step4: Install Python3 on the Ansible master

```bash
sudo apt install python3
```

### Install Ansible on Ubuntu 22.04 LTS

Step1: Install Ansible on Ubuntu 22.04 LTS

```bash
sudo apt install ansible -y
```
```bash
sudo apt install ansible-core -y
```

### Create an Inventory file in Ansible
To add inventory you can create a new directory or add in the default Ansible hosts file

```bash
cd /etc/ansible
sudo vi hosts
```
+ Now go to the host file inside the Ansible server and paste the public IP of the Jenkins
  
```bash
[local]#any name you want
Ip of Jenkins
```
+ install The Ansible plugin to integrate with Jenkins.
+ Add Credentials to invoke Ansible with Jenkins

Now write an Ansible playbook to create a docker image, tag it and push it to the docker hub, and finally, we will deploy it on a container using Ansible.

```bash
- name: docker build and push
  hosts: local  # Replace with the hostname or IP address of your target server
  become: yes  # Run tasks with sudo privileges

  tasks:
    - name: Update apt package cache
      apt:
        update_cache: yes   

    - name: Build Docker Image
      command: docker build -t petstore .
      args:
        chdir: /var/lib/jenkins/workspace/petstore

    - name: tag image
      command: docker tag petstore:latest erickay/petstore:latest 

    - name: Log in to Docker Hub
      community.docker.docker_login:
        registry_url: https://index.docker.io/v1/
        username: erickay
        password: dckr_pat_EbzrvA_OJDSPXRbSb-SFsHzytDE

    - name: Push image
      command: docker push erickay/petstore:latest

    - name: Run container
      command: docker run -d --name pet1 -p 8081:8080 erickay/petstore:latest
```

