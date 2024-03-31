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

__STEP1:__Create an Ubuntu(22.04) T2 Large Instance
