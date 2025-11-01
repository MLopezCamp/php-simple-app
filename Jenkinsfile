pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-cred')
        IMAGE_NAME = "mlopezcamp/php-simple-app"
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
                    // Obtiene el último commit del repo local
                    def currentCommit = sh(script: "git rev-parse HEAD", returnStdout: true).trim()

                    // Archivo para guardar el último commit desplegado
                    def commitFile = "${env.WORKSPACE}/.last_commit"

                    if (fileExists(commitFile)) {
                        def lastCommit = readFile(commitFile).trim()
                        if (currentCommit == lastCommit) {
                            echo "No hay cambios nuevos desde el último despliegue (${lastCommit})."
                            currentBuild.result = 'SUCCESS'
                            currentBuild.displayName = "Sin cambios"
                            skipRemainingStages()
                        } else {
                            echo "Cambios detectados. Último commit anterior: ${lastCommit}"
                        }
                    } else {
                        echo "Primer despliegue: no existe registro previo de commit."
                    }

                    // Guarda el commit actual para la próxima ejecución
                    writeFile file: commitFile, text: currentCommit
                }
            }
        }

        stage('Generate Tag') {
            when {
                expression { env.SKIP_BUILD != "true" }
            }
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
            when {
                expression { env.SKIP_BUILD != "true" }
            }
            steps {
                sh '''
                    echo "=== Construyendo imagen Docker ==="
                    docker build -t $IMAGE_NAME:$VERSION_TAG .
                    docker tag $IMAGE_NAME:$VERSION_TAG $IMAGE_NAME:latest
                '''
            }
        }

        stage('Login to DockerHub') {
            when {
                expression { env.SKIP_BUILD != "true" }
            }
            steps {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            }
        }

        stage('Push to DockerHub') {
            when {
                expression { env.SKIP_BUILD != "true" }
            }
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
            script {
                if (env.SKIP_BUILD == "true") {
                    echo "Sin cambios detectados — no se generó ni subió ninguna nueva imagen."
                } else {
                    echo "Pipeline completado con éxito"
                    echo "Se subieron las siguientes versiones:"
                    echo "-> $IMAGE_NAME:latest"
                    echo "-> $IMAGE_NAME:$VERSION_TAG"
                }
            }
        }
        failure {
            echo "Pipeline falló"
        }
    }
}
