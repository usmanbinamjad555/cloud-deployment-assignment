pipeline {
    agent any

    environment {
        PROJECT_NAME_CI = "salon"
        COMPOSE_FILE_CI = "docker-compose-ci.yaml"
        // Increased timeouts to prevent the pipeline from hanging during heavy builds
        DOCKER_CLIENT_TIMEOUT = '600'
        COMPOSE_HTTP_TIMEOUT = '600'
    }

    tools {
        git 'Default'
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo 'Checking out code from your repository...'
                // Using your specific GitHub URL
                git branch: 'main', url: 'https://github.com/usmanbinamjad555/cloud-deployment-assignment.git'
            }
        }

        stage('Stop Previous Containers') {
            steps {
                script {
                    echo 'Stopping any existing containers and cleaning up space...'
                    // We use || true so the pipeline doesn't fail if there are no containers to stop
                    sh "docker-compose -p ${PROJECT_NAME_CI} -f ${COMPOSE_FILE_CI} down || true"
                    sh "docker system prune -f"
                }
            }
        }

        stage('Build Images (using CI compose file)') {
            steps {
                dir("${env.WORKSPACE}") {
                    echo "Skipping manual build as YAML uses a base image. Pulling latest node image..."
                    sh "docker pull node:18-alpine"
                }
            }
        }

        stage('Run Containerized Application') {
            steps {
                echo "Starting containers on Port 8082..."
                sh "docker-compose -p ${PROJECT_NAME_CI} -f ${COMPOSE_FILE_CI} up -d"
                
                echo "Waiting 30 seconds for React to initialize..."
                sh "sleep 30"
                
                echo "Checking container status..."
                sh "docker-compose -p ${PROJECT_NAME_CI} -f ${COMPOSE_FILE_CI} ps"
            }
        }

        stage('Health Check') {
            steps {
                script {
                    echo 'Performing health check on Port 8082...'
                    // curl -f fails the stage if it gets a 404 or 500 error
                    sh """
                        sleep 10
                        curl -f http://localhost:8082/ || exit 1
                        echo "Health check passed!"
                    """
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished.'
            // Keep logs for troubleshooting
            sh "docker-compose -p ${PROJECT_NAME_CI} -f ${COMPOSE_FILE_CI} logs --tail=50 || true"
        }
        success {
            echo 'Pipeline successful! Application is running at http://54.206.37.103:8082'
        }
        failure {
            echo 'Pipeline failed. Cleaning up to save resources...'
            sh "docker-compose -p ${PROJECT_NAME_CI} -f ${COMPOSE_FILE_CI} down || true"
        }
    }
}
