stage('Test Website') {
    steps {
        sh '''
            docker rm -f website-test >/dev/null 2>&1 || true

            docker run -d --name website-test \
                my-static-website:${BUILD_NUMBER}

            for i in $(seq 1 10); do
                if docker exec website-test \
                    wget -qO- http://127.0.0.1 >/dev/null; then
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
