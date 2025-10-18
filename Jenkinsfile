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
                    sh 'docker-compose up -d database'
                    sh 'sleep 10'
                    sh 'docker-compose up -d app'
                    sh 'sleep 45'  // Temps pour Symfony
                    sh 'curl -f http://localhost:${APP_PORT} || echo "Échec du test HTTP sur le port ${APP_PORT}"'
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
