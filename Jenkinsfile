// Jenkinsfile
pipeline {
    agent {
        kubernetes {
            label 'dind-agent'
            defaultContainer 'jnlp'
        }
    }
    environment {
        DOCKER_IMAGE = "oddair/oddair:${env.BUILD_NUMBER}"
        DOCKERHUB_CREDENTIALS_ID = 'dockerhub-credentials' 
        GITHUB_CREDENTIALS_ID = 'github-pat'
    }
    stages {
        stage('1. Checkout Code') {
            steps {
                // VIKTIG ENDRING HER: Fra 'string' til 'usernamePassword'
                withCredentials([usernamePassword(credentialsId: GITHUB_CREDENTIALS_ID, usernameVariable: 'GIT_USERNAME_VAR', passwordVariable: 'GIT_PASSWORD_VAR')]) {
                    git credentialsId: GITHUB_CREDENTIALS_ID, url: 'https://github.com/OddAir/jenkins-k8s-app.git'
                }
            }
        }
        stage('2. Build and Push Docker Image') {
            steps {
                container('dind') {
                    withCredentials([usernamePassword(credentialsId: DOCKERHUB_CREDENTIALS_ID, usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh "docker login -u '${DOCKER_USERNAME}' -p '${DOCKER_PASSWORD}'"
                        sh "docker build -t '${DOCKER_IMAGE}' ."
                        sh "docker push '${DOCKER_IMAGE}'"
                    }
                }
            }
        }
        stage('3. Deploy to Kubernetes') {
            steps {
                container('jnlp') {
                    sh "sed -i 's|oddair/oddair:latest|${DOCKER_IMAGE}|g' deployment.yaml"
                    sh "kubectl apply -f deployment.yaml"
                }
            }
        }
    }
}
