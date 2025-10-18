pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'cronify_php'
        POSTGRES_DB = 'app'
        POSTGRES_USER = 'symfony'
        POSTGRES_PASSWORD = 'ChangeMe'
        POSTGRES_VERSION = '13-alpine'
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
                    // Construit l'image en spécifiant le contexte (dossier "docker/")
                    def customImage = docker.build(
                        "${DOCKER_IMAGE}:${env.BUILD_NUMBER}",
                        "docker/"  // Chemin vers le Dockerfile et le contexte
                    ).withEnv([
                        "POSTGRES_DB=${POSTGRES_DB}",
                        "POSTGRES_USER=${POSTGRES_USER}",
                        "POSTGRES_PASSWORD=${POSTGRES_PASSWORD}"
                    ])
                }
            }
        }


        stage('Test with Docker Compose') {
            steps {
                script {
                    // Lance les services avec docker-compose
                    sh 'docker-compose up -d database'
                    sh 'sleep 10'  // Attend que PostgreSQL soit prêt
                    sh 'docker-compose up -d app'

                    // Vérifie que l'application répond
                    sh 'sleep 15'  // Temps pour que l'app démarre
                    sh 'curl -f http://localhost:8080 || exit 1'
                }
            }
        }
    }

    post {
        always {
            // Arrête et nettoie les conteneurs
            sh 'docker-compose down -v || true'
        }
    }
}
