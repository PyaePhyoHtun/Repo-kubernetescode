pipeline {
    agent slave
    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    // Build Docker image
                    docker.build("pyaephyo28/capstone-app:latest")
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    // Push the Docker image to Docker Hub
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
                        docker.image("pyaephyo28/capstone-app:latest").push()
                    }
                }
            }
        }
        stage('Update Kubernetes Manifest') {
            steps {
                sh '''
                # Clone the Kubernetes repository with the main branch
                git clone --branch main https://github.com/PyaePhyoHtun/Repo-kubernetesmanifest.git
                cd Repo-kubernetesmanifest
                
                # Update Docker image tag in the deployment.yaml file
                sed -i 's|pyaephyo28/capstone-app:.*|pyaephyo28/capstone-app:latest|g' deployment.yaml
                
                # Configure Git for committing changes
                git config --global user.email "pyaephyohtun201@gmail.com"
                git config --global user.name "pyaephyotun"
                
                # Add, commit, and push changes to GitHub
                git add deployment.yaml
                git commit -m "Update image to latest"
                git push origin main
                '''
            }
        }
    }
}
