pipeline {
    agent any

    stages {

        stage('Build') {
            steps {
                sh '''
                    echo "=== BUILD STARTED ==="

                    /usr/bin/python3 -m venv venv

                    venv/bin/python -m pip install --upgrade pip

                    venv/bin/python -m pip install -r requirements.txt

                    echo "=== BUILD COMPLETED ==="
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    echo "=== TEST STARTED ==="

                    venv/bin/python -m pytest -v

                    echo "=== TEST COMPLETED ==="
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    echo "=== DEPLOYMENT STARTED ==="

                    echo "Deploying Flask application to staging environment..."

                    echo "=== DEPLOYMENT COMPLETED ==="
                '''
            }
        }
    }

    post {
        success {
            echo 'CI/CD Pipeline completed successfully!'
        }

        failure {
            echo 'CI/CD Pipeline failed!'
        }

        always {
            echo "Build completed: ${env.BUILD_NUMBER}"
        }
    }
}
