pipeline {
    agent none

    environment {
        IMAGE_NAME = "abode-web"
    }

    stages {

        stage('BUILD') {
            agent {
                label 'test'
            }

            steps {
                echo "Building application..."

                checkout scm

                sh '''
                    docker build \
                    -t ${IMAGE_NAME}:${BUILD_NUMBER} .
                '''
            }
        }

        stage('TEST') {
            agent {
                label 'test'
            }

            steps {
                echo "Testing application..."

                sh '''
                    docker rm -f abode-test 2>/dev/null || true

                    docker run -d \
                    --name abode-test \
                    -p 8082:80 \
                    ${IMAGE_NAME}:${BUILD_NUMBER}

                    slee

                    curl -f http://localhost:8082

                    docker rm -f abode-test
                '''
            }
        }

        stage('PRODUCTION') {
            when {
                branch 'main'
            }

            agent {
                label 'prod'
            }

            steps {
                echo "Deploying to production..."

                checkout scm

                sh '''
                    docker build \
                    -t ${IMAGE_NAME}:prod .
                '''

                sh '''
                    docker rm -f abode-production 2>/dev/null || true

                    docker run -d \
                    --name abode-production \
                    -p 80:80 \
                    --restart unless-stopped \
                    ${IMAGE_NAME}:prod
                '''

                sh '''
                    sleep 5
                    curl -f http://localhost
                '''
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully."
        }

        failure {
            echo "Pipeline failed. Check Console Output."
        }
    }
}
