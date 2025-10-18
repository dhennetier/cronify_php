pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'cronify_php'
        POSTGRES_DB = 'app'
        POSTGRES_USER = 'symfony'
        POSTGRES_PASSWORD = 'ChangeMe'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/dhennetier/cronify_php.git'
            }
        }

        stage('Debug: Vérifier les fichiers') {
            steps {
                script {
                    sh 'pwd'  // Affiche le répertoire courant
                    sh 'ls -la docker/'  // Vérifie le contenu de docker/
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Utilise le chemin absolu pour éviter toute ambiguïté
                    sh '''
                        cd docker/ && \
                        docker build -t ${DOCKER_IMAGE}:${env.BUILD_NUMBER} \
                            --build-arg POSTGRES_DB=${POSTGRES_DB} \
                            --build-arg POSTGRES_USER=${POSTGRES_USER} \
                            --build-arg POSTGRES_PASSWORD=${POSTGRES_PASSWORD} \
                            -f Dockerfile .
                    '''
                }
            }
        }

        stage('Test with Docker Compose') {
            steps {
                script {
                    sh 'docker-compose up -d database'
                    sh 'sleep 10'
                    sh 'docker-compose up -d app'
                    sh 'sleep 15'
                    sh 'curl -f http://localhost:8080 || exit 1'
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
