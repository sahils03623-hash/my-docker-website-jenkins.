
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
                bat 'docker build -t my-static-website:%BUILD_NUMBER% .'
            }
        }

        stage('Test Website') {
            steps {
                bat 'docker run -d --name website-test -p 8082:80 my-static-website:%BUILD_NUMBER%'
                bat 'powershell -NoProfile -Command "Invoke-WebRequest http://localhost:8082 -UseBasicParsing | Out-Null"'
            }
        }

        stage('Deploy Website') {
            steps {
                bat 'docker rm -f my-website 2>NUL || exit /b 0'
                bat 'docker run -d --restart unless-stopped -p 8081:80 --name my-website my-static-website:%BUILD_NUMBER%'
            }
        }
    }

    post {
        always {
            bat 'docker rm -f website-test 2>NUL || exit /b 0'
        }
        success {
            echo 'Website deployed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
