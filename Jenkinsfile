
pipeline {
    agent any

    triggers {
        pollSCM('H/1 * * * *')
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -f dockerfile -t my-static-website:${BUILD_NUMBER} .'
            }
        }

        stage('Test Website') {
            steps {
                sh '''
                    docker rm -f website-test >/dev/null 2>&1 || true

                    docker run -d --name website-test \
                        my-static-website:${BUILD_NUMBER}

                    for i in $(seq 1 10); do
                        if docker exec website-test \
                            wget -qO- http://127.0.0.1/ >/dev/null 2>&1; then
                            echo "Website test passed!"
                            exit 0
                        fi
                        sleep 2
                    done

                    echo "Website test failed!"
                    exit 1
                '''
            }
        }

        stage('Deploy Website') {
            steps {
                sh '''
                    docker rm -f my-website >/dev/null 2>&1 || true

                    docker run -d \
                        --restart unless-stopped \
                        -p 8081:80 \
                        --name my-website \
                        my-static-website:${BUILD_NUMBER}
                '''
            }
        }
    }

    post {
        always {
            sh 'docker rm -f website-test >/dev/null 2>&1 || true'
        }

        success {
            echo 'Website deployed successfully!'
        }

        failure {
            echo 'Pipeline failed!'
        }
    }
}

