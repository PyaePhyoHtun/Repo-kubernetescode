pipeline {
    agent {
        label 'Slave'  // Ensure the job runs on an Ubuntu agent
    }
    environment {
        DOCKER_HUB_CREDENTIALS = credentials('docker-hub-credentials')  // Jenkins credential ID for Docker Hub

    stages {
        stage('Checkout Code') {
            steps {
                echo 'Checking out code...'
                checkout scm  // Checkout the source code from the SCM (GitHub, GitLab, etc.)
            }
        }

        stage('Docker Login') {
            steps {
                echo 'Logging in to Docker Hub...'
                script {
                    sh "echo ${DOCKER_HUB_CREDENTIALS_PSW} | docker login -u ${DOCKER_HUB_CREDENTIALS_USR} --password-stdin"
                    // Logging in to Docker Hub using the Jenkins credentials
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                sh "docker build -t pyaephyo28/capstone-app:1.0 ."  // Build the Docker image
            }
        }

        stage('Push Docker Image') {
            steps {
                echo 'Pushing Docker image to Docker Hub...'
                sh "docker push pyaephyo28/capstone-app:1.0"  // Push the Docker image to Docker Hub
            }
        }
    }
}
