pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "pyaephyo28/capstone-app"
        DOCKER_TAG = "latest"
        GIT_REPO_MANIFEST = "https://github.com/PyaePhyolHum/Repo-kubernetesmanifest.git"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/PyaePhyoHtun/Repo-kubernetescode.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    bat 'docker build -t pyaephyo28/capstone-app:latest .'
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASSWORD')]) {
                        bat 'docker login -u %DOCKER_USER% -p %DOCKER_PASSWORD%'
                        bat 'docker push pyaephyo28/capstone-app:latest'
                    }
                }
            }
        }

        stage('Update Kubernetes Manifest') {
            steps {
                script {
                    bat """
                        git clone %GIT_REPO_MANIFEST%
                        cd Repo-kubernetesmanifest
                        powershell -Command "(Get-Content deployment.yaml) -replace 'image: pyaephyo28/capstone-app:.*', 'image: pyaephyo28/capstone-app:latest' | Set-Content deployment.yaml"
                        git add deployment.yaml
                        git commit -m "Update image to latest"
                        git pull origin main --rebase
                        git push origin main
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    bat 'kubectl apply -f Repo-kubernetesmanifest/deployment.yaml -n default'
                }
            }
        }
    }
}
