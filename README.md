# Kubernetes Project

## Step 1: Create Ubuntu EC2 & Install the Jenkins Server

Ubuntu 24.04 AMI
t2.medium
ALL TCP

### Update all software packages on Ubuntu server

sudo su - root
sudo apt-get update
sudo apt-get upgrade

### Install Java on Ubuntu server

sudo apt-get install default-jdk
java -version

### Jenkins Installation

Install Jenkins on Ubuntu server using the Link [Jenkins](https://www.jenkins.io/doc/book/installing/linux/#debianubuntu)
Access Jenkins Url with Public ip on port 8080
cat /var/lib/jenkins/secrets/initialAdminPassword
Install the Suggested Plugins
Setup UserName and Password

## Step 2: Get to know about Python Code and Create GitHub Repo

### Developer Actions

Create a Empty Github Repository
switch to root dir
mkdir DeveloperDir
cd DeveloperDir/
vi main.py
vi pod.yaml
vi requirements.txt
vi Dockerfile
vi form.html

### Push to files to Github

git init
git add .
git commit -m "First commit"
git remote add origin https://github.com/Iam-mithran/SaturdayProject
git push origin master
Generate a access Classic token from the GitHub Repo Settings

## Step 3: Create a Jenkins Pipeline Job And clone the Repo

### Create a Pipeline Job 

Install Stageview plugin from manageJenkins -> Plugins -> Available plugins

pipeline {
    agent any
    stages {
        stage('Pull Code From GitHub') {
            steps {
                git 'https://github.com/Iam-mithran/SaturdayProject.git'
            }
        }
    }
}

## Step 4: Containerize the Python app and publish to DockerHub

### Docker Installation

sudo apt install docker.io
Open DockerHub account
docker login

### Give Jenkins Permission to run sudo commands

vi /etc/sudoers
jenkins ALL=(ALL) NOPASSWD: ALL
exit
sudo su - root

### Run the three stages in Jenkinsfile

pipeline {
    agent any

    stages {
        stage('Pull Code From GitHub') {
            steps {
                git 'https://github.com/Iam-mithran/SaturdayProject.git'
            }
        }
        stage('Build the Docker image') {
            steps {
                sh 'sudo docker build -t weekendimage /var/lib/jenkins/workspace/weekendproj'
                sh 'sudo docker tag weekendimage iammithran/weekendimage:latest'
                sh 'sudo docker tag weekendimage iammithran/weekendimage:${BUILD_NUMBER}'
            }
        }
        stage('Push the Docker image') {
            steps {
                sh 'sudo docker image push iammithran/weekendimage:latest'
                sh 'sudo docker image push iammithran/weekendimage:${BUILD_NUMBER}'
            }
        }
    }
}

## Step 5: Create Kubernetes Cluster

### Install Kops
curl -Lo kops https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
chmod +x ./kops
sudo mv ./kops /usr/local/bin/

### Install kubectl

curl -Lo kubectl https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl

### AWS CLI Installation

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install --bin-dir /usr/local/bin --install-dir /usr/local/aws-cli --update
Create IAM user with Admin access policy and generate IAM access & Secret Key
aws configure
aws s3 ls

### Generate Keys

ssh-keygen

### Set Env Variables for Kops

export AWS_ACCESS_KEY_ID=
export AWS_SECRET_ACCESS_KEY=
export NAME=**mithran**.k8s.local
export KOPS_STATE_STORE=s3://**kops22042022**

### Create Kubernetes Cluster

kops create cluster --zones ap-south-1a ${NAME}
kops update cluster --name mithran.k8s.local --yes --admin

## Step 6: Deploy to kubernetes

### Add a new stage in Jenkins file

pipeline {
    agent any

    stages {
        stage('Pull Code From GitHub') {
            steps {
                git 'https://github.com/Iam-mithran/SaturdayProject.git'
            }
        }
        stage('Build the Docker image') {
            steps {
                sh 'sudo docker build -t weekendimage /var/lib/jenkins/workspace/weekendproj'
                sh 'sudo docker tag weekendimage iammithran/weekendimage:latest'
                sh 'sudo docker tag weekendimage iammithran/weekendimage:${BUILD_NUMBER}'
            }
        }
        stage('Push the Docker image') {
            steps {
                sh 'sudo docker image push iammithran/weekendimage:latest'
                sh 'sudo docker image push iammithran/weekendimage:${BUILD_NUMBER}'
            }
        }
        stage('Deploy on Kubernetes') {
            steps {
                sh 'sudo kubectl apply -f /var/lib/jenkins/workspace/weekendproj/pod.yaml'
                sh 'sudo kubectl rollout restart deployment loadbalancer-pod'
            }
        }
    }
}

### Configure Webhook in github & Jenkins

Github Repo Setting -> Webhooks -> http://43.205.203.11:8080/github-webhook/
Jenkins Job -> Build Triggers -> GitHub hook trigger for GITScm polling

### Make a Commit & Run the Deployment

make changes in the pod.yaml for correct image name
git -am commit "Test Run"
git push origin master
Check Deployment in Kubernetes