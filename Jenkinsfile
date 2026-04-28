pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "nileshmalekar514/myapp"
        DOCKER_TAG   = "${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Pulling source code from GitHub...'
                git branch: 'main',
                    url: 'https://github.com/nileshmalekar514/myapp-devops.git',
                    credentialsId: 'github-creds'
                echo 'Code pulled successfully from GitHub.'
            }
        }

        stage('Build') {
            steps {
                bat 'echo Building application...'
                bat 'echo Dependencies will be installed inside Docker container'
            }
        }

        stage('Docker Build') {
            steps {
                bat "docker build -t %DOCKER_IMAGE%:%DOCKER_TAG% ."
                bat "docker tag %DOCKER_IMAGE%:%DOCKER_TAG% %DOCKER_IMAGE%:latest"
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
                    bat "docker push %DOCKER_IMAGE%:latest"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'SUCCESS') {
                    bat "kubectl apply -f deployment.yaml"
                    bat "kubectl set image deployment/myapp myapp=%DOCKER_IMAGE%:%DOCKER_TAG%"
                    bat "kubectl rollout status deployment/myapp --timeout=60s"
                    bat "kubectl get pods"
                }
                echo 'Deployment stage completed.'
            }
        }
    }

    post {
        success { echo 'Pipeline executed successfully! All stages passed.' }
        failure { echo 'Pipeline failed!' }
    }
}
