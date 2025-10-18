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
                        docker-compose build --no-cache app
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
                    sh 'sleep 45'

                    // Vérifications
                    echo '=== Extensions PHP chargées ==='
                    sh 'docker-compose exec app php -m'

                    echo '=== Drivers PDO disponibles ==='
                    sh 'docker-compose exec app php -r "print_r(PDO::getAvailableDrivers());"'

                    echo '=== Variables d\'environnement PostgreSQL ==='
                    sh 'docker-compose exec app env | grep POSTGRES'

                    echo '=== Contenu du fichier .env ==='
                    sh 'docker-compose exec app cat .env'

                    echo '=== Logs de l\'application ==='
                    sh 'docker-compose logs app'

                    echo '=== Test de connexion HTTP ==='
                    sh 'curl -v http://localhost:${APP_PORT} || echo "Échec de la connexion au port ${APP_PORT}"'
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
