pipeline {
    agent any
    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("pyaephyo28/capstone-app")
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
                        docker.image("pyaephyo28/capstone-app").push()
                    }
                }
            }
        }
        stage('Update Kubernetes Manifest') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github-token-credentials', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                    sh '''
                    git clone https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/PyaePhyoHtun/Repo-kubernetesmanifest.git
                    cd Repo-kubernetesmanifest
                    sed -i 's|pyaephyo28/capstone-app:.*|pyaephyo28/capstone-app:latest|g' deployment.yaml
                    git config --global user.email "pyaephyohtun201@gmail.com"
                    git config --global user.name "Jenkins"
                    git add deployment.yaml
                    git commit -m "Update image to latest"
                    git push origin main
                    '''
                }
            }
        }
    }
}
