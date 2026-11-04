
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
                        -p 8083:80 \
                        my-static-website:${BUILD_NUMBER}

                    curl --fail --retry 10 --retry-connrefused \
                        --retry-delay 2 \
                        http://host.docker.internal:8083
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

