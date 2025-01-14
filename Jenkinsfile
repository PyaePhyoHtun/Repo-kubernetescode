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
                script {
                    sh """
                        git clone ${GIT_REPO_MANIFEST}
                        cd Repo-kubernetesmanifest
                        sed -i 's|image: ${DOCKER_IMAGE}:.*|image: ${DOCKER_IMAGE}:${DOCKER_TAG}|g' deployment.yaml
                        git add deployment.yaml
                        git commit -m "Update image to ${DOCKER_TAG}"
                        git push origin main
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh """
                        kubectl apply -f Repo-kubernetesmanifest/deployment.yaml -n default
                    """
                }
            }
        }
    }
}
