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
                    bat """
                    echo "=== Cloning Repo-kubernetesmanifest ==="
                    if exist Repo-kubernetesmanifest rmdir /s /q Repo-kubernetesmanifest
                    git clone https://%GIT_USERNAME%:%GIT_PASSWORD%@github.com/PyaePhyoHtun/Repo-kubernetesmanifest.git Repo-kubernetesmanifest
                    cd Repo-kubernetesmanifest
        
                    echo "=== Deleting and recreating deployment.yaml ==="
                    if exist deployment.yaml del deployment.yaml
                    echo apiVersion: apps/v1 > deployment.yaml
                    echo kind: Deployment >> deployment.yaml
                    echo metadata: >> deployment.yaml
                    echo "  name: capstone-app" >> deployment.yaml
                    echo spec: >> deployment.yaml
                    echo "  replicas: 3" >> deployment.yaml
                    echo "  selector:" >> deployment.yaml
                    echo "    matchLabels:" >> deployment.yaml
                    echo "      app: capstone-app" >> deployment.yaml
                    echo "  template:" >> deployment.yaml
                    echo "    metadata:" >> deployment.yaml
                    echo "      labels:" >> deployment.yaml
                    echo "        app: capstone-app" >> deployment.yaml
                    echo "    spec:" >> deployment.yaml
                    echo "      containers:" >> deployment.yaml
                    echo "      - name: capstone-app" >> deployment.yaml
                    echo "        image: ${env.DOCKER_IMAGE}:${env.DOCKER_TAG}" >> deployment.yaml
                    echo "        imagePullPolicy: Always" >> deployment.yaml
                    echo "        ports:" >> deployment.yaml
                    echo "        - containerPort: 8080" >> deployment.yaml
        
                    echo "=== Updated deployment.yaml ==="
                    type deployment.yaml
        
                    echo "=== Configuring Git ==="
                    git config --global user.email "pyaephyohtun201@gmail.com"
                    git config --global user.name "%GIT_USERNAME%"
        
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
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
