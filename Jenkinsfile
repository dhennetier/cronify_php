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
                    // Lance les services
                    sh 'docker-compose up -d database'
                    sh 'sleep 10'
                    sh 'docker-compose up -d app'
                    sh 'sleep 45'  // Temps supplémentaire pour Symfony

                    // Vérifie l'état des conteneurs
                    sh 'docker-compose ps'

                    // Affiche les logs de l'app
                    sh 'docker-compose logs app'

                    // Teste la connectivité réseau depuis le conteneur database
                    sh 'docker-compose exec database curl -f http://app:80 || echo "App non joignable depuis database"'

                    // Teste depuis l'hôte (si le port est mappé)
                    sh 'curl -v http://localhost:${APP_PORT} || echo "Port ${APP_PORT} non accessible"'
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
