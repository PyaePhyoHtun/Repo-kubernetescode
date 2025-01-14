pipeline {
    agent any

    environment {
        IMAGE_NAME = "pyaephyo28/capstone-app"
        IMAGE_TAG = "latest"
        GIT_CREDENTIALS = 'github-token-credentials' // Ensure this credential exists in Jenkins
        DOCKER_HUB_CREDENTIALS = 'docker-hub-credentials' // Ensure this credential exists in Jenkins
    }

    stages {
        stage('Checkout Code') {
            steps {
                script {
                    checkout scm
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
                }
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', DOCKER_HUB_CREDENTIALS) {
                        docker.image("${IMAGE_NAME}:${IMAGE_TAG}").push()
                    }
                }
            }
        }

        stage('Update Kubernetes Manifest in GitHub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: GIT_CREDENTIALS, usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                        sh '''
                        rm -rf Repo-kubernetesmanifest
                        git clone https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/PyaePhyoHtun/Repo-kubernetesmanifest.git
                        cd Repo-kubernetesmanifest
                        sed -i 's|pyaephyo28/capstone-app:.*|pyaephyo28/capstone-app:latest|g' deployment.yaml
                        git config --global user.email "pyaephyohtun201@gmail.com"
                        git config --global user.name "PyaePhyoHtun"
                        git add deployment.yaml
                        git commit -m "Update image to latest"
                        git push origin main
                        '''
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed. Please check the logs for errors."
        }
    }
}
