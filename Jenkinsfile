```groovy
pipeline {
    agent any

    environment {
        VENV = "${WORKSPACE}/venv"
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo 'Creating Python virtual environment...'

                sh '''
                    python3 -m venv "$VENV"

                    . "$VENV/bin/activate"

                    python --version
                    pip --version

                    pip install --upgrade pip

                    if [ -f requirements.txt ]; then
                        pip install -r requirements.txt
                    else
                        echo "requirements.txt not found."
                        pip install Flask pytest
                    fi
                '''
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'

                sh '''
                    . "$VENV/bin/activate"

                    if [ -d tests ] || ls test_*.py >/dev/null 2>&1; then
                        pytest -v
                    else
                        echo "No pytest tests found."
                        echo "Running application syntax check..."

                        if [ -f app.py ]; then
                            python -m py_compile app.py
                        elif [ -f application.py ]; then
                            python -m py_compile application.py
                        else
                            echo "No Flask application file found."
                            exit 1
                        fi
                    fi
                '''
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying application to staging...'

                sh '''
                    . "$VENV/bin/activate"

                    # Stop any previous Flask process
                    if [ -f flask.pid ]; then
                        kill $(cat flask.pid) 2>/dev/null || true
                        rm -f flask.pid
                    fi

                    # Start Flask application
                    export FLASK_APP=app.py

                    nohup flask run \
                        --host=0.0.0.0 \
                        --port=5000 \
                        > flask.log 2>&1 &

                    echo $! > flask.pid

                    sleep 5

                    echo "Flask application started."
                    cat flask.log || true
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                echo 'Verifying deployment...'

                sh '''
                    sleep 2

                    if curl -f http://localhost:5000; then
                        echo ""
                        echo "Application is running successfully!"
                    else
                        echo ""
                        echo "Application failed to respond."
                        cat flask.log || true
                        exit 1
                    fi
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'

            mail to: 'lambhate.ankit@gmail.com',
                 subject: "Jenkins Pipeline Success: ${env.JOB_NAME} Build #${env.BUILD_NUMBER}",
                 body: "The pipeline executed successfully. Check Jenkins for details."
        }

        failure {
            echo 'Pipeline failed!'

            mail to: 'lambhate.ankit@gmail.com',
                 subject: "Jenkins Pipeline FAILED: ${env.JOB_NAME} Build #${env.BUILD_NUMBER}",
                 body: "The pipeline failed. Check Jenkins console output for details."
        }
    }
}
