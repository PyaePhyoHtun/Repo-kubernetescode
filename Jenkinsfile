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
                    bat '''
                    powershell -Command "(Get-Content repo-manifest/deployment.yaml) -replace 'pyaephyo28/capstone-app:.*', 'pyaephyo28/capstone-app:latest' | Set-Content Repo-kubernetesmanifest/deployment.yaml"
                    cd Repo-kubernetesmanifest
                    git config --global user.email "pyaephyohtun201@gmail.com"
                    git config --global user.name "%GIT_USERNAME%"
                    git add deployment.yaml
                    git commit -m "Update image to latest"
                    git push https://%GIT_USERNAME%:%GIT_PASSWORD%@github.com/PyaePhyoHtun/Repo-kubernetesmanifest.git main
                    '''
                }
            }
        }
    }
}
