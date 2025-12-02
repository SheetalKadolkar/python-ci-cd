pipeline {
    agent any

    environment {
        IMAGE_NAME = "sheetalkadolkar/python-ci-cd:${BUILD_ID}"
        CONTAINER_NAME = "python-app-${BUILD_ID}"
    }

    stages {

        /* ---------------- PREPARE ---------------- */
        stage('Prepare') {
            steps {
                echo 'Checking project files...'
                sh 'ls -l'
                sh 'cat app.py'
                sh 'cat requirements.txt'
            }
        }

        /* ---------------- INSTALL & TEST ---------------- */
        stage('Install & Run Tests') {
            steps {
                echo 'Installing dependencies & running unit tests...'
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                    python test/test.py

                '''
            }
        }

        /* ---------------- DOCKER BUILD ---------------- */
        stage('Build Docker Image') {
            steps {
                echo "Building Docker image..."
                sh "docker build -t ${IMAGE_NAME} ."
            }
        }

        /* ---------------- RUN CONTAINER ---------------- */
        stage('Run Docker Container') {
            steps {
                echo 'Running Docker container...'

                sh '''
                    docker stop ${CONTAINER_NAME} || true
                    docker rm ${CONTAINER_NAME} || true
                    docker run -d -p 5000:5000 --name ${CONTAINER_NAME} ${IMAGE_NAME}
                '''
            }
        }

        /* ---------------- TEST APPLICATION ---------------- */
        stage('Test Application') {
            steps {
                echo 'Testing application endpoints...'
                sh '''
                    sleep 10

                    echo "Testing / endpoint"
                    curl -f http://localhost:5000/ || exit 1

                    echo "Testing /hi endpoint"
                    curl -f http://localhost:5000/hi || exit 1
                '''
            }
        }
    }

    post {
        always {
            echo 'Cleaning up...'
            sh '''
                docker stop ${CONTAINER_NAME} || true
                docker rm ${CONTAINER_NAME} || true
                docker system prune -f || true
            '''
        }

        success {
            echo '✅ Pipeline completed successfully!'
        }

        failure {
            echo '❌ Pipeline failed!'
        }
    }
}
