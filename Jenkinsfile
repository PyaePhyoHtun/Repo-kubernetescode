pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "pyaephyo28/capstone-app"
        DOCKER_TAG = "latest"
        GIT_REPO_CODE = "https://github.com/PyaePhyoHtun/Repo-kubernetescode.git"
        GIT_REPO_MANIFEST = "https://github.com/PyaePhyolHum/Repo-kubernetesmanifest.git"
    }

    stages {
        stage('Checkout Application Code') {
            steps {
                git branch: 'main', url: env.GIT_REPO_CODE
            }
        }

        stage('Check for Changes in app.py') {
            steps {
                script {
                    // Check if app.py has changed in the latest commit
                    def changes = bat(script: 'git diff --name-only HEAD HEAD~1', returnStdout: true).trim()
                    if (changes.contains('app.py')) {
                        echo "Changes detected in app.py. Proceeding with build and deployment."
                        env.APP_CHANGED = true
                    } else {
                        echo "No changes detected in app.py. Skipping build and deployment."
                        env.APP_CHANGED = false
                    }
                }
            }
        }

        stage('Build Docker Image') {
            when {
                expression { env.APP_CHANGED == 'true' }
            }
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}")
                }
            }
        }

        stage('Push Docker Image') {
            when {
                expression { env.APP_CHANGED == 'true' }
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
                        docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").push()
                    }
                }
            }
        }

        stage('Checkout Kubernetes Manifest') {
            when {
                expression { env.APP_CHANGED == 'true' }
            }
            steps {
                script {
                    // Clone the Kubernetes manifest repository
                    bat '''
                    if exist Repo-kubernetesmanifest rmdir /s /q Repo-kubernetesmanifest
                    git clone %GIT_REPO_MANIFEST% Repo-kubernetesmanifest
                    '''
                }
            }
        }

        stage('Update Kubernetes Manifest') {
            when {
                expression { env.APP_CHANGED == 'true' }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'github-token-credentials', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                    bat '''
                    cd Repo-kubernetesmanifest
                    echo "Updating deployment.yaml..."
                    powershell -Command "(Get-Content deployment.yaml) -replace 'image: .*', 'image: pyaephyo28/capstone-app:latest' | Set-Content deployment.yaml"
                    echo "Updated deployment.yaml."
                    git config --global user.email "pyaephyohtun201@gmail.com"
                    git config --global user.name "%GIT_USERNAME%"
                    git add deployment.yaml
                    git commit -m "Update image to latest"
                    echo "Pulling latest changes from GitHub..."
                    git pull https://%GIT_USERNAME%:%GIT_PASSWORD%@github.com/PyaePhyolHum/Repo-kubernetesmanifest.git main
                    echo "Pushing changes to GitHub..."
                    git push https://%GIT_USERNAME%:%GIT_PASSWORD%@github.com/PyaePhyolHum/Repo-kubernetesmanifest.git main
                    echo "Changes pushed to GitHub."
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            when {
                expression { env.APP_CHANGED == 'true' }
            }
            steps {
                script {
                    bat 'kubectl apply -f Repo-kubernetesmanifest/deployment.yaml -n default'
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
