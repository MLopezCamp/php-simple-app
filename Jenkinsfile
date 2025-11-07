pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-cred')
        IMAGE_NAME = "mlopezcamp/php-simple-app"
        LAST_COMMIT_FILE = ".last_commit"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/MLopezCamp/php-simple-app.git'
            }
        }

        stage('Detect Changes') {
            steps {
                script {
                    def currentCommit = sh(script: "git rev-parse HEAD", returnStdout: true).trim()
                    def lastCommit = fileExists(LAST_COMMIT_FILE) ? readFile(LAST_COMMIT_FILE).trim() : ""

                    if (currentCommit == lastCommit) {
                        echo "No hay cambios nuevos desde el último despliegue (${currentCommit})."
                        currentBuild.result = 'SUCCESS'
                        return
                    } else {
                        echo "Se detectaron cambios. Último commit desplegado: ${lastCommit}, nuevo commit: ${currentCommit}"
                        writeFile file: LAST_COMMIT_FILE, text: currentCommit
                    }
                }
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
            echo "Se subieron las siguientes versiones (si hubo cambios):"
            echo "→ $IMAGE_NAME:latest"
            echo "→ $IMAGE_NAME:$VERSION_TAG"
        }
        failure {
            echo "Pipeline falló"
        }
    }
}
