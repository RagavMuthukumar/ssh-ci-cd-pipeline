# Setting Up a Jenkins CI/CD Pipeline for Dockerized Java Applications with Multiple Java Versions

This project demonstrates how to set up a Jenkins CI/CD pipeline to build and deploy Dockerized Java applications with support for multiple Java versions.

## Prerequisites

- Jenkins installed on a master node (Java 17 required).
- EC2 instances configured:
  - **Jenkins Master (jenkins-master EC2)**: Installed Jenkins and Java 17.
  - **Java 8 Instance (java-8 EC2)**: Installed Java 1.8.0, Git, Maven, and Docker.
  
## Initial Setup

### Jenkins Setup
1. Installed Jenkins on the master node.
2. Configured the Jenkins instance with its public IP.
3. Installed the **SSH Agent Plugin** on Jenkins.

### SSH Key Setup
1. Generated an SSH key pair using `ssh-keygen`.
2. Added the public key to the `authorized_keys` on the Java 8 instance.
3. Configured Jenkins credentials with the SSH private key to allow secure connections.

## Pipeline Configuration

A Jenkins pipeline was created with the following stages:

1. **Clone Repository**: 
   - Clones the Git repository from GitHub to the Java 8 instance.

2. **Build on Remote EC2**:
   - Runs a Maven build on the Java 8 instance to compile the project.

3. **Build Docker Image**:
   - Builds a Docker image on the Java 8 instance using the `Dockerfile` in the repository.

4. **Build Docker Image and Run Container on Remote EC2**:
   - Rebuilds the Docker image (if necessary), removes previous containers, and deploys a new container.

## Pipeline Script

```groovy
pipeline {
    agent any
    stages {
        stage('Clone Repository') {
            steps {
                sshagent(['ssh-agent']) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no ec2-user@<JAVA_8_INSTANCE_IP> "git clone https://github.com/<USERNAME>/Java-Springboot.git /home/ec2-user/Java-Springboot"
                    '''
                }
            }
        }
        stage('Build on Remote EC2') {
            steps {
                sshagent(['ssh-agent']) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no ec2-user@<JAVA_8_INSTANCE_IP> "cd /home/ec2-user/Java-Springboot && mvn clean install"
                    '''
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                sshagent(['ssh-agent']) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no ec2-user@<JAVA_8_INSTANCE_IP> "cd /home/ec2-user/Java-Springboot && sudo docker build -t java-springboot-image ."
                    '''
                }
            }
        }
        stage('Build Docker Image and Run Container on Remote EC2') {
            steps {
                sshagent(['ssh-agent']) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no ec2-user@<JAVA_8_INSTANCE_IP> "
                    cd /home/ec2-user/Java-Springboot
                    sudo docker build -t java-springboot-image .
                    sudo docker stop java-springboot-container || true
                    sudo docker rm java-springboot-container || true
                    sudo docker run -d --name java-springboot-container -p 8081:8080 java-springboot-image
                    "
                    '''
                }
            }
        }
    }
}
