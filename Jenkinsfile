pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "pyaephyo28/capstone-app"
        DOCKER_TAG = "latest"
        GIT_REPO_CODE = "https://github.com/PyaePhyoHtun/Repo-kubernetescode.git"
        GIT_REPO_MANIFEST = "https://github.com/PyaePhyoHtun/Repo-kubernetesmanifest.git"
    }

    stages {
        stage('Checkout Application Code') {
            steps {
                git branch: 'main', url: env.GIT_REPO_CODE
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
                        docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").push()
                    }
                }
            }
        }

        stage('Update Kubernetes Manifest') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github-token-credentials', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                    bat '''
                    if exist Repo-kubernetesmanifest rmdir /s /q Repo-kubernetesmanifest
                    git clone https://%GIT_USERNAME%:%GIT_PASSWORD%@github.com/PyaePhyoHtun/Repo-kubernetesmanifest.git Repo-kubernetesmanifest
                    cd Repo-kubernetesmanifest
                    powershell -Command "(Get-Content deployment.yaml) -replace 'IMAGE_PLACEHOLDER', '${DOCKER_IMAGE}' | Set-Content deployment.yaml"
                    powershell -Command "(Get-Content deployment.yaml) -replace 'TAG_PLACEHOLDER', '${DOCKER_TAG}' | Set-Content deployment.yaml"
                    git config --global user.email "pyaephyohtun201@gmail.com"
                    git config --global user.name "%GIT_USERNAME%"
                    git add deployment.yaml
                    git commit -m "Update image to ${DOCKER_IMAGE}:${DOCKER_TAG}"
                    git pull origin main
                    git push origin main
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
