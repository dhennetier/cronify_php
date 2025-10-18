pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'cronify_php'
        POSTGRES_DB = 'app'
        POSTGRES_USER = 'symfony'
        POSTGRES_PASSWORD = 'ChangeMe'
        APP_PORT = '9000'
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
                    sh """
                        docker build -t ${DOCKER_IMAGE}:${env.BUILD_NUMBER} \
                            --build-arg POSTGRES_DB=${POSTGRES_DB} \
                            --build-arg POSTGRES_USER=${POSTGRES_USER} \
                            --build-arg POSTGRES_PASSWORD=${POSTGRES_PASSWORD} \
                            -f docker/Dockerfile .
                    """
                }
            }
        }

        stage('Test with Docker Compose') {
            steps {
                script {
                    // Lance les services en arrière-plan
                    sh 'docker-compose up -d database'
                    sh 'sleep 10'  // Attend que PostgreSQL soit prêt
                    sh 'docker-compose up -d app'
                    sh 'sleep 30'  // Attend que Symfony démarre complètement

                    // Vérifie que le conteneur est en bonne santé
                    sh 'docker-compose ps'

                    // Teste depuis le réseau du conteneur (plus fiable)
                    sh 'docker-compose exec app curl -f http://localhost:80 || exit 1'

                    // Alternative : Teste depuis l'hôte (si le port est mappé)
                    sh 'curl -f http://localhost:${APP_PORT} || exit 1'
                }
            }
        }
    }

    post {
        always {
            sh 'docker-compose down -v || true'
        }
    }
}
