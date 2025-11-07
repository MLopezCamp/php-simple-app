pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-cred')
        IMAGE_NAME = "mlopezcamp/php-simple-app"
        REPO_URL = "https://github.com/MLopezCamp/php-simple-app.git"
        TRACK_FILE = "${env.JENKINS_HOME}/last_commit_php_simple_app"
    }

    stages {
        stage('Verificar cambios en repositorio remoto') {
            steps {
                script {
                    echo "=== Verificando si el repositorio remoto tiene nuevos commits ==="

                    // Obtener el último commit remoto de GitHub
                    def remoteCommit = sh(
                        script: "git ls-remote ${REPO_URL} refs/heads/main | cut -f1",
                        returnStdout: true
                    ).trim()

                    echo "Último commit remoto: ${remoteCommit}"

                    // Leer el commit anterior almacenado (si existe)
                    def previousCommit = ""
                    if (fileExists(TRACK_FILE)) {
                        previousCommit = readFile(TRACK_FILE).trim()
                        echo "Último commit registrado: ${previousCommit}"
                    } else {
                        echo "No existe registro previo, se considerará como primera ejecución."
                    }

                    // Comparar commits
                    if (remoteCommit == previousCommit) {
                        echo "Sin cambios detectados, no se subirá la imagen."
                        currentBuild.result = 'SUCCESS'
                        // Terminar pipeline anticipadamente
                        error("Sin cambios detectados — pipeline detenido.")
                    } else {
                        echo "Cambios detectados, se procederá a construir y subir la nueva imagen."
                        writeFile file: TRACK_FILE, text: remoteCommit
                    }
                }
            }
        }

        stage('Checkout') {
            steps {
                git branch: 'main', url: "${REPO_URL}"
            }
        }

        stage('Generar tag') {
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
                    echo "=== Construyendo imagen Docker ==="
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
        }
        failure {
            echo "Pipeline finalizado sin subir imagen (posiblemente sin cambios)"
        }
    }
}
