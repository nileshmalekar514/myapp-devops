pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "nileshmalekar514/myapp"
        DOCKER_TAG   = "${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/nileshmalekar514/myapp-devops.git',
                    credentialsId: 'github-creds'
            }
        }

        stage('Build') {
            steps {
                bat 'echo Building application...'
                bat 'pip install -r requirements.txt'
            }
        }

        stage('Docker Build') {
            steps {
                bat "docker build -t %DOCKER_IMAGE%:%DOCKER_TAG% ."
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS')]) {
                    bat "docker login -u %DOCKER_USER% -p %DOCKER_PASS%"
                    bat "docker push %DOCKER_IMAGE%:%DOCKER_TAG%"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                bat "kubectl set image deployment/myapp myapp=%DOCKER_IMAGE%:%DOCKER_TAG% --record"
                bat "kubectl rollout status deployment/myapp"
            }
        }
    }

    post {
        success { echo 'Pipeline executed successfully!' }
        failure { echo 'Pipeline failed!' }
    }
}
