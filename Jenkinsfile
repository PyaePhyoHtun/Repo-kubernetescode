pipeline {
    agent any
    stages {
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
                sh '''
                git clone https://github.com/PyaePhyoHtun/Repo-kubernetesmanifest.git
                cd Repo-kubernetesmanifest
                sed -i 's|pyaephyo28/capstone-app:.*|pyaephyo28/capstone-app:latest|g' deployment.yaml
                git config --global user.email "pyaephyohtun201@gmail.com"
                git config --global user.name "pyaephyotun"
                git add deployment.yaml
                git commit -m "Update image to latest"
                git push origin main  // Ensure you're pushing to the correct branch (main)
                '''
            }
        }
    }
}
