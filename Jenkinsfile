pipeline {
    agent {
        kubernetes {
            // Definerer en bygge-pod med en Docker-container
            // og gir den tilgang til Docker-verktøyene på din maskin.
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: docker
    image: docker:24-git
    command:
    - cat
    tty: true
    volumeMounts:
    - name: dockersock
      mountPath: /var/run/docker.sock
  volumes:
  - name: dockersock
    hostPath:
      path: /var/run/docker.sock
'''
        }
    }
    stages {
        stage('Build Docker Image') {
            steps {
                // Kjører stegene inne i Docker-containeren
                container('docker') {
                    script {
                        def imageName = "oddair/my-first-pipeline-app:${env.BUILD_NUMBER}"
                        
                        echo "Bygger Docker-image: ${imageName}"
                        sh "docker build -t ${imageName} ."
                        
                        echo "Bygging fullført. Viser lokale Docker-images:"
                        sh "docker images"
                    }
                }
            }
        }
    }
}
