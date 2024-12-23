# Setting Up a Jenkins CI/CD Pipeline for Dockerized Java Applications with Multiple Java Versions on AWS EC2

This repository contains the configuration and setup for automating the deployment of a Java application using Jenkins, Docker, and AWS EC2 instances. The setup involves using multiple Java versions (Java 8 and Java 17) to build and deploy a Java Spring Boot application.

## Table of Contents
1. [Initial Setup](#initial-setup)
2. [Jenkins Setup](#jenkins-setup)
3. [SSH Key Setup](#ssh-key-setup)
4. [Pipeline Configuration](#pipeline-configuration)
5. [Pipeline Script](#pipeline-script)
6. [Result](#result)

---

## Initial Setup

1. **Jenkins Master (jenkins-masters EC2):**
   - Installed **Jenkins** and **Java 17** on the EC2 instance.

2. **Java 8 Instance (java 8 EC2):**
   - Installed **Java 1.8.0**, **Git**, **Maven**, and **Docker** on the EC2 instance.

---

## Jenkins Setup

1. **Connect to Jenkins Master:**
   - Connect to the Jenkins master instance using its public IP address and complete the Jenkins setup.

2. **Install SSH Agent Plugin:**
   - In Jenkins, install the **SSH Agent Plugin** to manage SSH keys.

---

## SSH Key Setup

1. **Generate SSH Key Pair:**
   - Run `ssh-keygen` to generate an SSH key pair.

2. **Add Public Key to Java 8 Instance:**
   - Add the generated **public key** to the `authorized_keys` file on the **Java 8 instance** for SSH access.

3. **Configure Jenkins Credentials:**
   - In Jenkins, configure SSH credentials using the **private key** for authenticating with the Java 8 instance.

---

## Pipeline Configuration

1. **Pipeline Project:**
   - A Jenkins pipeline project is created to automate the following tasks:
     - Clone the Git repository
     - Build the project using **Java 8** or **Java 17** (depending on the requirement)
     - Build a Docker image
     - Run the Docker container

---

## Pipeline Script

The Jenkins pipeline script is written in **Groovy** and consists of the following stages:

```groovy
pipeline {
    agent any

    stages {
        stage('Clone Repository') {
            steps {
                sshagent(['ssh-agent']) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no ec2-user@3.109.186.65 "git clone https://github.com/Sushmaa123/Java-Springboot.git /home/ec2-user/Java-Springboot"
                    '''
                }
            }
        }
        stage('Build on Remote EC2 (Java 8 or Java 17)') {
            steps {
                sshagent(['ssh-agent']) {
                    sh '''
                    # Choose Java 8 or Java 17 based on the requirement
                    ssh -o StrictHostKeyChecking=no ec2-user@3.109.186.65 "cd /home/ec2-user/Java-Springboot && mvn clean install -Djava.version=1.8"
                    '''
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                sshagent(['ssh-agent']) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no ec2-user@3.109.186.65 "cd /home/ec2-user/Java-Springboot && sudo docker build -t java-springboot-image ."
                    '''
                }
            }
        }
        stage('Build Docker Image and Run Container on Remote EC2') {
            steps {
                sshagent(['ssh-agent']) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no ec2-user@3.109.186.65 "
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
