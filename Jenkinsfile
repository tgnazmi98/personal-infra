pipeline {
    agent any

    environment {
        COMPOSE_FILE = '/opt/traefik/docker-compose.yml'
        DEPLOY_DIR = '/opt/traefik'
    }

    stages {
        stage('Pull From Repo') {
            steps {
                dir("${DEPLOY_DIR}") {
                    sh "git pull origin main"
                }
            }
        }

        stage('Deploy Traefik') {
            steps {
                sh "docker compose -f ${COMPOSE_FILE} pull"
                sh "docker compose -f ${COMPOSE_FILE} up -d"
            }
        }

        stage('Health Check') {
            steps {
                retry(10) {
                    sleep 15
                    sh """
                        CONTAINER_ID=\$(docker compose -f ${COMPOSE_FILE} ps -q traefik)
                        STATUS=\$(docker inspect --format='{{.State.Health.Status}}' "\$CONTAINER_ID")
                        echo "Traefik health status: \$STATUS"
                        [ "\$STATUS" = "healthy" ]
                    """
                }
            }
        }
    }

    post {
        failure {
            echo 'Traefik deployment failed!'
        }
        success {
            echo 'Traefik deployed successfully.'
        }
    }
}
