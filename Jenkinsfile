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
                    echo "Cloning the Kubernetes manifest repository..."
                    if exist Repo-kubernetesmanifest rmdir /s /q Repo-kubernetesmanifest
                    git clone https://%GIT_USERNAME%:%GIT_PASSWORD%@github.com/PyaePhyoHtun/Repo-kubernetesmanifest.git Repo-kubernetesmanifest
                    cd Repo-kubernetesmanifest
                    echo "Updating deployment.yaml..."
                    powershell -Command "(Get-Content deployment.yaml) -replace 'image: .*', 'image: pyaephyo28/capstone-app:latest' | Set-Content deployment.yaml"
                    echo "Updated deployment.yaml."
                    git config --global user.email "pyaephyohtun201@gmail.com"
                    git config --global user.name "%GIT_USERNAME%"
                    git add deployment.yaml
                    git commit -m "Update image to latest"
                    echo "Pulling latest changes from GitHub..."
                    git pull https://%GIT_USERNAME%:%GIT_PASSWORD%@github.com/PyaePhyoHtun/Repo-kubernetesmanifest.git main
                    echo "Pushing changes to GitHub..."
                    git push https://%GIT_USERNAME%:%GIT_PASSWORD%@github.com/PyaePhyoHtun/Repo-kubernetesmanifest.git main
                    echo "Changes pushed to GitHub."
                    '''
                }
            }
        }
    }
}
