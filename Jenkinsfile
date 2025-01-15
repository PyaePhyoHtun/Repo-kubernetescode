pipeline {
    agent any

    stages {
        stage('Checkout Application Code') {
            steps {
                git 'https://github.com/PyaePhyoHtun/Repo-kubernetescode.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("pyaephyo28/capstone-app:latest")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
                        docker.image("pyaephyo28/capstone-app:latest").push()
                    }
                }
            }
        }

        stage('Update Kubernetes Manifest') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github-token-credentials', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                    bat '''
                        echo "=== Cloning Repo-kubernetesmanifest ==="
                        if exist Repo-kubernetesmanifest rmdir /s /q Repo-kubernetesmanifest
                        git clone https://%GIT_USERNAME%:%GIT_PASSWORD%@github.com/PyaePhyoHtun/Repo-kubernetesmanifest.git Repo-kubernetesmanifest
                        cd Repo-kubernetesmanifest
                        echo "=== Deleting and recreating deployment.yaml ==="
                        if exist deployment.yaml del deployment.yaml
                        (
                            echo apiVersion: apps/v1
                            echo kind: Deployment
                            echo metadata:
                            echo "  name: capstone-app"
                            echo spec:
                            echo "  replicas: 3"
                            echo "  selector:"
                            echo "    matchLabels:"
                            echo "      app: capstone-app"
                            echo "  template:"
                            echo "    metadata:"
                            echo "      labels:"
                            echo "        app: capstone-app"
                            echo "    spec:"
                            echo "      containers:"
                            echo "      - name: capstone-app"
                            echo "        image: pyaephyo28/capstone-app:latest"
                            echo "        imagePullPolicy: Always"
                            echo "        ports:"
                            echo "        - containerPort: 8080"
                        ) > deployment.yaml
                        echo "=== Updated deployment.yaml ==="
                        type deployment.yaml
                        echo "=== Configuring Git ==="
                        git config --global user.email "pyaephyohtun201@gmail.com"
                        git config --global user.name "PyaePhyoHtun"
                        echo "=== Forcefully adding and committing changes ==="
                        git add deployment.yaml
                        git commit --allow-empty -m "Update image to pyaephyo28/capstone-app:latest"
                        git pull origin main
                        git push origin main
                    '''
                }
            }
        }

        stage('Restart Deployment') {
            steps {
                script {
                    bat 'kubectl rollout restart deployment/capstone-app'
                }
            }
        }
    }

    post {
        failure {
            echo 'Pipeline failed!'
        }
    }
}
