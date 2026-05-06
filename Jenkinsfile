pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'Sana26072007/laravel-app'
        DOCKER_CREDENTIALS_ID = 'docker-hub-credentials'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', 
                    url: 'https://github.com/Sana26072007/ТВОЙ_РЕПОЗИТОРИЙ.git'
            }
        }

        stage('Setup Environment') {
            steps {
                script {
                    sh 'cp .env.production .env'
                    sh 'docker run --rm -v $(pwd):/app -w /app php:8.2-cli php artisan key:generate --force --no-interaction'
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${BUILD_NUMBER}")
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('', DOCKER_CREDENTIALS_ID) {
                        docker.image("${DOCKER_IMAGE}:${BUILD_NUMBER}").push()
                    }
                }
            }
        }

        stage('Deploy to Production') {
            steps {
                script {
                    sh '''
                        docker-compose down --volumes --remove-orphans || true
                        docker-compose up -d --build
                        docker-compose exec app composer install --no-dev --optimize-autoloader
                        docker-compose exec app php artisan optimize:clear
                        docker-compose exec app php artisan optimize
                        docker-compose exec app php artisan migrate --force
                    '''
                }
            }
        }
    }

    post {
        success {
            echo '✅ Laravel application deployed!'
        }
        failure {
            echo '❌ Deployment failed!'
        }
    }
}
