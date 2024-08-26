# CI/CD Pipeline with Jenkins, SonarQube, and Docker on AWS EC2

This project demonstrates how to set up a complete CI/CD pipeline that automatically pulls code from a GitHub repository, performs static code analysis using SonarQube, and deploys the code to a Docker container. All components are hosted on separate AWS EC2 instances.

## Table of Contents

1. [Introduction](#introduction)
2. [Architecture](#architecture)
3. [Prerequisites](#prerequisites)
4. [Setup](#setup)
   - [Step 1: Launch EC2 Instances](#step-1-launch-ec2-instances)
   - [Step 2: Install and Configure Jenkins](#step-2-install-and-configure-jenkins)
   - [Step 3: Install and Configure SonarQube](#step-3-install-and-configure-sonarqube)
   - [Step 4: Configure Docker](#step-4-configure-docker)
   - [Step 5: Set Up CI/CD Pipeline in Jenkins](#step-5-set-up-cicd-pipeline-in-jenkins)
5. [Testing](#testing)
6. [Troubleshooting](#troubleshooting)

## Introduction

This repository contains a complete setup for a CI/CD pipeline using Jenkins, SonarQube, and Docker. The pipeline is designed to:

- Automatically pull code from a GitHub repository.
- Analyze code quality using SonarQube.
- Deploy the application to a Docker container running on an AWS EC2 instance.

## Architecture

The architecture involves the following components:

- **Jenkins**: Automates the build, test, and deployment process.
- **SonarQube**: Performs static code analysis to ensure code quality.
- **Docker**: Runs the application in a containerized environment.
- **AWS EC2 Instances**: Hosts Jenkins, SonarQube, and Docker.

![Network Diagrams (1)](https://github.com/user-attachments/assets/0e508808-0469-4465-9211-ad380620bec9)


## Prerequisites

- AWS account with EC2 permissions
- GitHub account
- Basic knowledge of Jenkins, SonarQube, Docker, and AWS EC2
- AWS CLI installed and configured

## Setup

### Step 1: Launch EC2 Instances

1. **Login to your AWS Management Console.**
2. **Launch three EC2 instances**:
   - **Jenkins Instance**: Choose an instance type (e.g., t2.medium) and install Jenkins.
   - **SonarQube Instance**: Choose an instance type (e.g., t2.medium) and install SonarQube.
   - **Docker Instance**: Choose an instance type (e.g., t2.medium) and install Docker.

3. **Security Groups**: Ensure the security groups allow traffic between instances on necessary ports (e.g., 8080 for Jenkins, 9000 for SonarQube, and 80/443 for Docker).

### Step 2: Install and Configure Jenkins

1. **SSH into the Jenkins instance**.
2. **Install Jenkins**:
   ```bash
   sudo apt update
   sudo apt install openjdk-11-jdk
   wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
   sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary > /etc/apt/sources.list.d/jenkins.list'
   sudo apt update
   sudo apt install jenkins
   ```
### Access Jenkins

Open a browser and navigate to `http://<JENKINS_PUBLIC_IP>:8080`. Follow the setup wizard to complete the installation.

### Install Required Plugins

Go to "Manage Jenkins" -> "Manage Plugins" and install:
- Git Plugin
- Docker Plugin
- SonarQube Scanner Plugin

### Configure Jenkins

Set up Jenkins to connect to GitHub and Docker. Configure SonarQube integration under "Manage Jenkins" -> "Configure System".
### Step 3: Install and Configure SonarQube
-SSH into the SonarQube instance.

-Install SonarQube:
```
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.7.1.62043.zip
unzip sonarqube-9.7.1.62043.zip
cd sonarqube-9.7.1.62043/bin/linux-x86-64
./sonar.sh start
```
### Access SonarQube

Open a browser and navigate to `http://<SONARQUBE_PUBLIC_IP>:9000`. Complete the initial setup and create a new project.
## Step 4: Configure Docker

SSH into the Docker instance.

### Install Docker:

```
sudo apt update
sudo apt install docker.io
sudo systemctl start docker
sudo systemctl enable docker
```
## Step 5: Set Up CI/CD Pipeline in Jenkins

### Create a new Jenkins pipeline project.

### Configure GitHub repository
- You can use you own .html file for your website.

In the pipeline settings, link your GitHub repository.

### Define the Jenkinsfile

Create a `Jenkinsfile` in your repository with the following content:

```
pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/your-repository.git'
            }
        }
        stage('Build') {
            steps {
                sh './build.sh'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn sonar:sonar'
                }
            }
        }
        stage('Docker Build & Deploy') {
            steps {
                sh 'docker build -t your-image .'
                sh 'docker run -d -p 80:80 your-image'
            }
        }
    }
}
```

# How to Configure Docker in Jenkins

## Set up Docker Host

Jenkins uses a REST API for communicating with Docker. The following configuration steps on the Docker host ensure that the Jenkins controller can connect properly.

1. Use a tool such as Nmap to check if the relevant ports are open. Docker Remote API uses port 4243, while ports 32768 to 60999 are assigned to Jenkins for connecting with Docker containers.

2. Open the `docker.service` file in a text editor:

    ```bash
    sudo nano /lib/systemd/system/docker.service
    ```

   Find the line starting with `ExecStart` and replace it with the following:

    ```bash
    ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:4243 -H unix:///var/run/docker.sock
    ```

3. Reload the Docker daemon:

    ```bash
    sudo systemctl daemon-reload
    ```

4. Restart the Docker service:

    ```bash
    sudo service docker restart
    ```

5. Test the connection with the curl command:

    ```bash
    curl http://localhost:4243/version
    ```

   The command outputs the Docker version details.

## Create Docker Image

Docker images require configuration to work as build agents in Jenkins. When you create Docker images for this purpose, make sure they contain:

- All the build dependencies, such as Git or Java.
- Jenkins login credentials.
- The `sshd` service mapped to port 22 for SSH-based communication.

## Install Docker Plugin

Jenkins has a Docker plugin that enables communication with Docker hosts. To install the plugin in Jenkins, do the following:
<img width="400" alt="home-screen-manage-jenkins-item-configure-docker-jenkins-update" src="https://github.com/user-attachments/assets/7dbe2cc1-123c-41c4-9171-ea3191115ee0">

1. Select **Manage Jenkins** in the menu on the left side of the Jenkins dashboard.

2. Click **Manage Plugins** in the Manage Jenkins window.

3. Select the **Available** tab in the Plugin Manager window.

4. Type **Docker** in the search field, and select the box next to the Docker plugin that appears in the search results.

5. Click the **Download now and install after restart** button.
<img width="350" alt="installing-docker-plugin-jenkins-ui-update" src="https://github.com/user-attachments/assets/92d33a8b-440b-40b8-b1ba-9c1f9b6c7f18">

6. When all the necessary plugin components download, select the box at the bottom of the screen to restart Jenkins.

   Jenkins restarts automatically when the installation completes.

## Configure Docker Build Agent in Jenkins

Configure the Docker Build Agent to perform jobs in the Manage Jenkins window of the Jenkins dashboard.

1. Select the **Manage Nodes and Clouds** item in the System Configuration section.
   <img width="350" alt="manage-jenkins-section-manage-nodes-and-clouds-item-jenkins-update" src="https://github.com/user-attachments/assets/d23b946c-98e9-4c92-986d-4ca41ee37b2e">

3. Click **Configure Clouds** in the menu on the left side.

4. Expand the **Add a new cloud** list and select **Docker**.
<img width="300" alt="name-docker-host-uri-enabled-jenkins-ui-update" src="https://github.com/user-attachments/assets/ee16c01d-4c5e-4660-b1c4-e73d14be454d">

5. Provide the name and URI for the Docker host in the relevant fields. Enable the host by selecting the box.

6. Select the **Expose DOCKER_HOST** box.

7. Click the **Docker Agent templates** button to open additional configuration options.

8. Provide the label for identifying the host, and enable the agent by selecting the **Enabled** option.
<img width="300" alt="labels-java-docker-slave-jenkins-ui-update" src="https://github.com/user-attachments/assets/f5250e81-82b5-490c-b7f0-d6cafc5f88f7">

9. Name the Docker template and provide the path to the Docker image.

10. Specify the home folder for the Jenkins user you created.
<img width="300" alt="remote-file-system-root-connect-method-jenkins-ui-update" src="https://github.com/user-attachments/assets/75d49fc9-e5d2-4104-94a2-fd8c47bba5af">

11. Choose **Connect with SSH** from the list in the **Connect method** section.

    Additional SSH configuration options appear.

12. Select **Use configured SSH credentials** in the SSH key section. Provide the credentials you set up for the image in the field that appears below.

13. Select **Non verifying Verification Strategy** in the Host Key Verification Strategy section.
<img width="300" alt="ssh-key-credentials-configure-docker-jenkins-ui-update" src="https://github.com/user-attachments/assets/5e906f2d-d16a-42d8-8510-f070a9d92cbb">

14. Click **Save** when you finish configuring the host.

   The Docker build agent setup is complete.

## Test Docker Containers in Jenkins

Once the Docker host is fully configured in Jenkins, test it by creating a Freestyle project.

1. Select **New Item** from the menu on the left side of the main dashboard.

2. Name the job, select the **Freestyle project** option and click **OK**.
<img width="300" alt="test-job-freestyle-project-docker-jenkins-ui-update" src="https://github.com/user-attachments/assets/d1429d93-9ed2-4476-9527-013623e34d21">

3. Select **Restrict where this project can be run** option and type the Docker host label in the field below.

4. Add a simple build step. For example, use the `echo` command to display a message in the shell. Click **Save** when you finish configuring the job.
<img width="300" alt="execute-shell-test-job-jenkins-ui-update" src="https://github.com/user-attachments/assets/93f6cf3f-8c1c-42c8-9240-d77dc5adee8e">

5. Select **Build Now** in the menu on the left side of the project window.

   The job initiates. Track the job's progress in the Build History section on the left side.

6. When the job completes, click the item in the Build History section. The Build window appears.


7. Select **Console Output** in the menu on the left side.

8. Check if the console output contains the command you scheduled for execution.
   <img width="375" alt="console-output-test-job-docker-jenkins-ui-update" src="https://github.com/user-attachments/assets/6a5c193c-fa39-4661-816b-a7d3387312e1">

   The console additionally prints a **SUCCESS** message, indicating the build worked.

## Testing

1. **Push a change to your GitHub repository.**
2. **Verify Jenkins**: Check that the pipeline runs successfully and that SonarQube performs analysis.
3. **Verify Docker Deployment**: Ensure that your application is correctly deployed by accessing the Docker container.


## Troubleshooting

- **Jenkins Connectivity Issues**: Check security groups and network ACLs to ensure proper communication between Jenkins and other services.
- **SonarQube Analysis Problems**: Verify SonarQube configuration and check logs for errors.
- **Docker Deployment Issues**: Ensure Docker containers are running and accessible. Check Docker logs for errors.
