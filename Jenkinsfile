pipeline {
    agent any
    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("pyaephyo28/capstone-app:latest")  // Use your actual Docker Hub username and repo
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
                        docker.image("pyaephyo28/capstone-app:latest").push()  // Use your actual Docker Hub username and repo
                    }
                }
            }
        }
        stage('Update Kubernetes Manifest') {
            steps {
                sh '''
                git clone https://github.com/PyaePhyoHtun/Repo-kubernetesmanifest.git  // Replace with your repo URL
                cd Repo-kubernetesmanifest
                sed -i 's|pyaephyo28/capstone-app:.*|pyaephyo28/capstone-app:latest|g' deployment.yaml  // Update with your Docker image name
                git config --global user.email "pyaephyohtun201@gmail.com"  // Replace with Jenkins email
                git config --global user.name "pyaephyotun"  // Replace with Jenkins name
                git add deployment.yaml
                git commit -m "Update image to latest"
                git push origin main
                '''
            }
        }
    }
}
