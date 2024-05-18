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
