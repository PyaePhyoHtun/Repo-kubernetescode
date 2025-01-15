pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "pyaephyo28/capstone-app"  // Docker image name
        DOCKER_TAG = "latest"                     // Docker image tag
        GIT_REPO_CODE = "https://github.com/PyaePhyoHtun/Repo-kubernetescode.git"  // Application code repo
        GIT_REPO_MANIFEST = "https://github.com/PyaePhyoHtun/Repo-kubernetesmanifest.git"  // Manifest repo
    }

    stages {
        // Stage 1: Checkout Application Code
        stage('Checkout Application Code') {
            steps {
                git branch: 'main', url: env.GIT_REPO_CODE
            }
        }

        // Stage 2: Build Docker Image
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${env.DOCKER_IMAGE}:${env.DOCKER_TAG}")
                }
            }
        }

        // Stage 3: Push Docker Image to Docker Hub
        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
                        docker.image("${env.DOCKER_IMAGE}:${env.DOCKER_TAG}").push()
                    }
                }
            }
        }

        stage('Update Kubernetes Manifest') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github-token-credentials', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                    bat """
                    echo "=== Cloning Repo-kubernetesmanifest ==="
                    if exist Repo-kubernetesmanifest rmdir /s /q Repo-kubernetesmanifest
                    git clone https://%GIT_USERNAME%:%GIT_PASSWORD%@github.com/PyaePhyoHtun/Repo-kubernetesmanifest.git Repo-kubernetesmanifest
                    cd Repo-kubernetesmanifest
        
                    echo "=== Updating deployment.yaml ==="
                    powershell -Command "(Get-Content deployment.yaml) -replace 'image: .*', 'image: ${env.DOCKER_IMAGE}:${env.DOCKER_TAG}' | Set-Content deployment.yaml"
        
                    echo "=== Configuring Git ==="
                    git config --global user.email "pyaephyohtun201@gmail.com"
                    git config --global user.name "PyaePhyoHtun"
        
                    echo "=== Forcefully adding and committing changes ==="
                    git add deployment.yaml
                    git commit --allow-empty -m "Update image to ${env.DOCKER_IMAGE}:${env.DOCKER_TAG}"
                    git pull origin main
                    git push origin main
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully! Argo CD will now sync the changes.'
        }
        failure {
            echo 'Pipeline failed! Check logs for errors.'
        }
    }
}
