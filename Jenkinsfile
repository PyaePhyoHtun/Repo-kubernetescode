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
                    if (isUnix()) {
                        sh 'docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .'
                    } else {
                        bat 'docker build -t %DOCKER_IMAGE%:%DOCKER_TAG% .'
                    }
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASSWORD')]) {
                        if (isUnix()) {
                            sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USER --password-stdin'
                            sh 'docker push ${DOCKER_IMAGE}:${DOCKER_TAG}'
                        } else {
                            bat 'docker login -u %DOCKER_USER% -p %DOCKER_PASSWORD%'
                            bat 'docker push %DOCKER_IMAGE%:%DOCKER_TAG%'
                        }
                    }
                }
            }
        }

        stage('Update Kubernetes Manifest') {
            steps {
                script {
                    if (isUnix()) {
                        sh '''
                            if [ -d "Repo-kubernetesmanifest" ]; then
                                rm -rf Repo-kubernetesmanifest
                            fi
                            git clone ${GIT_REPO_MANIFEST}
                            cd Repo-kubernetesmanifest
                            sed -i "s|image: ${DOCKER_IMAGE}:.*|image: ${DOCKER_IMAGE}:${DOCKER_TAG}|g" deployment.yaml
                            git add deployment.yaml
                            git commit -m "Update image to ${DOCKER_TAG}"
                            git pull origin main --rebase
                            git push origin main
                        '''
                    } else {
                        bat """
                            if exist Repo-kubernetesmanifest rmdir /s /q Repo-kubernetesmanifest
                            git clone %GIT_REPO_MANIFEST%
                            cd Repo-kubernetesmanifest
                            powershell -Command "(Get-Content deployment.yaml) -replace 'image: ${DOCKER_IMAGE}:.*', 'image: ${DOCKER_IMAGE}:${DOCKER_TAG}' | Set-Content deployment.yaml"
                            git add deployment.yaml
                            git commit -m "Update image to ${DOCKER_TAG}"
                            git pull origin main --rebase
                            git push origin main
                        """
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    if (isUnix()) {
                        sh 'kubectl apply -f Repo-kubernetesmanifest/deployment.yaml -n default'
                    } else {
                        bat 'kubectl apply -f Repo-kubernetesmanifest/deployment.yaml -n default'
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline succeeded!'
            // Add notifications for success (e.g., email, Slack)
        }
        failure {
            echo 'Pipeline failed!'
            // Add notifications for failure (e.g., email, Slack)
        }
    }
}
