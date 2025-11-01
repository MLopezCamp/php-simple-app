pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-cred')  // crea credencial en Jenkins con tu usuario/pass de DockerHub
        IMAGE_NAME = "mlopezcamp/php-simple-app"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/MLopezCamp/php-simple-app.git'
            }
        }

        stage('Generate Tag') {
            steps {
                script {
                    def GIT_COMMIT = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                    def DATE_TAG = sh(script: "date +%Y%m%d-%H%M%S", returnStdout: true).trim()
                    def VERSION_TAG = "${DATE_TAG}-${GIT_COMMIT}"
                    env.VERSION_TAG = VERSION_TAG
                    echo "Versión generada: ${VERSION_TAG}"
                }
            }
        }


        stage('Build Docker Image') {
            steps {
                sh '''
                    echo "=== Construyendo imagen ==="
                    docker build -t $IMAGE_NAME:$VERSION_TAG .
                    docker tag $IMAGE_NAME:$VERSION_TAG $IMAGE_NAME:latest
                '''
            }
        }

        stage('Login to DockerHub') {
            steps {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            }
        }

        stage('Push to DockerHub') {
            steps {
                sh '''
                    echo "=== Subiendo imagen a DockerHub ==="
                    docker push $IMAGE_NAME:$VERSION_TAG
                    docker push $IMAGE_NAME:latest
                '''
            }
        }
    }

    post {
        always {
            echo "=== Limpieza final ==="
            sh 'docker system prune -f || true'
        }
        success {
            echo "Pipeline completado con éxito"
            echo "Se subieron las siguientes versiones:"
            echo "→ $IMAGE_NAME:latest"
            echo "→ $IMAGE_NAME:$VERSION_TAG"
        }
        failure {
            echo "Pipeline falló"
        }
    }
}
