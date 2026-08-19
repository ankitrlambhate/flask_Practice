pipeline {

    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate

                    python -m pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    . venv/bin/activate
                    pytest -v
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    echo "Deploying Flask application..."

                    pkill -f "python.*app.py" || true

                    . venv/bin/activate

                    nohup python app.py > flask.log 2>&1 &

                    sleep 5

                    echo "Flask application deployed successfully"
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
