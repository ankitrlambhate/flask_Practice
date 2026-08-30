pipeline {

    agent any

    environment {
        APP_DIR = "/var/lib/jenkins/workspace/flask_app_ankit"
        PYTHON_PACKAGES = "/var/lib/jenkins/workspace/flask_app_ankit/python_packages"
        APP_PORT = "5000"
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
                echo 'Building Flask application...'

                sh '''
                    set -e

                    echo "======================================"
                    echo "Python Environment"
                    echo "======================================"

                    whoami
                    python3 --version
                    which python3

                    echo ""
                    echo "Cleaning previous build..."

                    rm -rf "$PYTHON_PACKAGES"
                    mkdir -p "$PYTHON_PACKAGES"

                    echo ""
                    echo "Installing dependencies..."

                    if [ -f requirements.txt ]; then
                        python3 -m pip install \
                            --target="$PYTHON_PACKAGES" \
                            -r requirements.txt
                    else
                        echo "ERROR: requirements.txt not found"
                        exit 1
                    fi

                    echo ""
                    echo "Verifying Flask installation..."

                    PYTHONPATH="$PYTHON_PACKAGES" python3 -c \
                        "import flask; print('Flask version:', flask.__version__)"

                    echo ""
                    echo "Build completed successfully."
                '''
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'

                withCredentials([
                    string(
                        credentialsId: 'mongo-uri',
                        variable: 'MONGO_URI'
                    ),
                    string(
                        credentialsId: 'secret-key',
                        variable: 'SECRET_KEY'
                    )
                ]) {

                    sh '''
                        set -e

                        export PYTHONPATH="$PYTHON_PACKAGES"

                        echo "======================================"
                        echo "Running Flask Tests"
                        echo "======================================"

                        echo "Python version:"
                        python3 --version

                        echo ""
                        echo "Checking project tests..."

                        if [ -f test_app.py ]; then

                            echo "test_app.py found."

                            MONGO_URI="$MONGO_URI" \
                            SECRET_KEY="$SECRET_KEY" \
                            python3 -m pytest test_app.py -v

                        elif [ -d tests ]; then

                            echo "tests directory found."

                            MONGO_URI="$MONGO_URI" \
                            SECRET_KEY="$SECRET_KEY" \
                            python3 -m pytest tests -v

                        else

                            echo "WARNING: No tests found."
                            echo "Skipping tests."

                        fi

                        echo ""
                        echo "Tests completed successfully."
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying Flask application...'

                withCredentials([
                    string(
                        credentialsId: 'mongo-uri',
                        variable: 'MONGO_URI'
                    ),
                    string(
                        credentialsId: 'secret-key',
                        variable: 'SECRET_KEY'
                    )
                ]) {

                    sh '''
                        set -e

                        echo "======================================"
                        echo "Deploying Flask Application"
                        echo "======================================"

                        export PYTHONPATH="$PYTHON_PACKAGES"
                        export MONGO_URI="$MONGO_URI"
                        export SECRET_KEY="$SECRET_KEY"

                        # Stop previous Flask application if running
                        if [ -f flask.pid ]; then

                            OLD_PID=$(cat flask.pid)

                            if kill -0 "$OLD_PID" 2>/dev/null; then
                                echo "Stopping previous Flask application..."
                                kill "$OLD_PID" || true
                                sleep 2
                            fi

                            rm -f flask.pid
                        fi

                        # Also stop anything using port 5000
                        if command -v fuser >/dev/null 2>&1; then
                            fuser -k ${APP_PORT}/tcp 2>/dev/null || true
                            sleep 2
                        fi

                        echo "Starting Flask application..."

                        nohup python3 app.py > flask.log 2>&1 &

                        FLASK_PID=$!

                        echo "$FLASK_PID" > flask.pid

                        echo "Flask PID: $FLASK_PID"

                        sleep 5

                        echo ""
                        echo "Checking Flask process..."

                        if kill -0 "$FLASK_PID" 2>/dev/null; then
                            echo "Flask application started successfully."
                        else
                            echo "ERROR: Flask application failed to start."
                            echo ""
                            echo "Flask log:"
                            cat flask.log || true
                            exit 1
                        fi
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                echo 'Verifying Flask deployment...'

                sh '''
                    set -e

                    echo "Waiting for application..."
                    sleep 3

                    echo "Testing localhost:${APP_PORT}..."

                    if curl -f --max-time 10 "http://127.0.0.1:${APP_PORT}/"; then
                        echo ""
                        echo "======================================"
                        echo "Deployment verification successful!"
                        echo "======================================"
                    else
                        echo ""
                        echo "ERROR: Application verification failed."
                        echo ""
                        echo "Flask log:"
                        cat flask.log || true
                        exit 1
                    fi
                '''
            }
        }
    }

    post {

        success {
            echo '======================================'
            echo 'Pipeline completed successfully!'
            echo '======================================'

            mail(
                to: 'YOUR_EMAIL@example.com',
                subject: "SUCCESS: Flask Pipeline #${BUILD_NUMBER}",
                body: """
Flask CI/CD Pipeline completed successfully.

Job: ${JOB_NAME}
Build: #${BUILD_NUMBER}
Status: SUCCESS

Application has been deployed successfully.
"""
            )
        }

        failure {
            echo '======================================'
            echo 'Pipeline FAILED!'
            echo '======================================'

            sh '''
                if [ -f flask.pid ]; then

                    PID=$(cat flask.pid)

                    if kill -0 "$PID" 2>/dev/null; then
                        echo "Stopping Flask application..."
                        kill "$PID" || true
                    fi

                fi
            '''

            mail(
                to: 'YOUR_EMAIL@example.com',
                subject: "FAILED: Flask Pipeline #${BUILD_NUMBER}",
                body: """
Flask CI/CD Pipeline failed.

Job: ${JOB_NAME}
Build: #${BUILD_NUMBER}
Status: FAILURE

Please check the Jenkins console output for details.
"""
            )
        }

        always {
            echo '======================================'
            echo 'Pipeline execution finished.'
            echo '======================================'
        }
    }
}

