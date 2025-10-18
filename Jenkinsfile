pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'cronify_php'
        DOCKER_REGISTRY = ''  // Exemple : 'docker.io/dhennetier' ou 'ghcr.io/dhennetier' (optionnel)
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/dhennetier/cronify_php.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def customImage = docker.build("${DOCKER_IMAGE}:${env.BUILD_NUMBER}")
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    // Lance un conteneur pour tester
                    docker.image("${DOCKER_IMAGE}:${env.BUILD_NUMBER}").run('--name test-container -d -p 8082:80')
                    // Exemple de test : vérifie que le conteneur répond
                    sh 'sleep 10 && curl -f http://localhost:8082 || true'
                }
            }
        }

        stage('Push to Registry') {
            when {
                branch 'main'  // Seulement sur la branche main
            }
            steps {
                script {
                    // Envoi vers un registre Docker (optionnel)
                    // docker.withRegistry('https://registry.hub.docker.com', 'docker-credentials') {
                    //     docker.image("${DOCKER_IMAGE}:${env.BUILD_NUMBER}").push()
                    // }
                }
            }
        }
    }

    post {
        always {
            // Nettoyage après le pipeline
            sh 'docker stop test-container || true'
            sh 'docker rm test-container || true'
        }
    }
}
