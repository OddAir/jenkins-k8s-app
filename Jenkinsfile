
// Jenkinsfile
pipeline {
    // Definerer at jobben skal kjøre på vår "dind-agent" mal i Kubernetes.
    agent {
        kubernetes {
            label 'dind-agent'
            defaultContainer 'jnlp'
        }
    }

    // Definerer variabler vi bruker i pipelinen.
    environment {
        // Ditt Docker Hub repo er satt inn her:
        DOCKER_IMAGE = "oddair/oddair:${env.BUILD_NUMBER}"
        DOCKERHUB_CREDENTIALS_ID = 'dockerhub-credentials' 
    }

    stages {
        stage('1. Checkout Code') {
            steps {
                // Din GitHub Repo URL er satt inn her:
                git 'https://github.com/OddAir/jenkins-k8s-app.git' 
            }
        }

        stage('2. Build and Push Docker Image') {
            steps {
                // Vi velger å kjøre disse kommandoene i 'dind'-containeren.
                container('dind') {
                    // Henter Docker Hub-credentials trygt fra Jenkins.
                    withCredentials([usernamePassword(credentialsId: DOCKERHUB_CREDENTIALS_ID, usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        // Logger inn, bygger og pusher imaget.
                        sh "docker login -u '${DOCKER_USERNAME}' -p '${DOCKER_PASSWORD}'"
                        sh "docker build -t '${DOCKER_IMAGE}' ."
                        sh "docker push '${DOCKER_IMAGE}'"
                    }
                }
            }
        }

        stage('3. Deploy to Kubernetes') {
            steps {
                // Kjører i default-containeren ('jnlp').
                container('jnlp') {
                    // Erstatter placeholder-imaget med det nye imaget vi nettopp bygget.
                    // Docker Hub repo brukes her:
                    sh "sed -i 's|oddair/oddair:latest|${DOCKER_IMAGE}|g' deployment.yaml"
                    sh "kubectl apply -f deployment.yaml"
                }
            }
        }
    }
}
