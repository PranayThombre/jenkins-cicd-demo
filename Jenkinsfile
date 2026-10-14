pipeline {
    agent any

    environment {
        IMAGE_NAME = 'jenkins-cicd-demo'
        CONTAINER_NAME = 'cicd-website'
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Source code is available in the workspace'
                sh 'ls -la'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build \
                      -t ${IMAGE_NAME}:${BUILD_NUMBER} .
                '''
            }
        }

        stage('Test Application') {
            steps {
                sh '''
                    set -eu

                    TEST_CONTAINER="cicd-test-${BUILD_NUMBER}"
                    IMAGE="${IMAGE_NAME}:${BUILD_NUMBER}"

                    cleanup() {
                        docker rm -f "$TEST_CONTAINER" \
                          >/dev/null 2>&1 || true
                    }
                    trap cleanup EXIT

                    docker run -d \
                      --name "$TEST_CONTAINER" \
                      -p 127.0.0.1:18082:80 \
                      "$IMAGE"

                    for i in $(seq 1 15); do
                        if curl -fsS http://127.0.0.1:18082 \
                            | grep -q "My First CI/CD Project"; then
                            echo "Application test passed"
                            exit 0
                        fi
                        sleep 2
                    done

                    echo "Application test failed"
                    exit 1
                '''
            }
        }

        stage('Deploy Application') {
            steps {
                sh '''
                    set -eu

                    # Stop the previous application container.
                    docker rm -f "$CONTAINER_NAME" \
                      >/dev/null 2>&1 || true

                    # Deploy the tested image.
                    docker run -d \
                      --name "$CONTAINER_NAME" \
                      --restart unless-stopped \
                      -p 8082:80 \
                      "${IMAGE_NAME}:${BUILD_NUMBER}"
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    set -eu

                    for i in $(seq 1 15); do
                        if curl -fsS http://127.0.0.1:8082 \
                            | grep -q "My First CI/CD Project"; then
                            echo "Deployment successful"
                            exit 0
                        fi
                        sleep 2
                    done

                    echo "Deployment verification failed"
                    exit 1
                '''
            }
        }
    }

    post {
        success {
            echo 'CI/CD pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed. Check Console Output.'
        }
    }
}