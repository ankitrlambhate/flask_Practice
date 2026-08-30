pipeline {
    agent any

    environment {
        VENV = "${WORKSPACE}/venv"
    }

    stages {

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
                    echo "Checking Python venv support..."
                    python3 -c "import ensurepip; print('ensurepip: OK')"
                    python3 -c "import venv; print('venv module: OK')"

                    echo ""
                    echo "Creating virtual environment..."
                    rm -rf "$VENV"
                    python3 -m venv "$VENV"

                    echo ""
                    echo "Activating virtual environment..."
                    . "$VENV/bin/activate"

                    echo "Python:"
                    python --version

                    echo "Pip:"
                    pip --version

                    echo ""
                    echo "Upgrading pip..."
                    pip install --upgrade pip

                    echo ""
                    echo "Installing application dependencies..."

                    if [ -f requirements.txt ]; then
                        echo "requirements.txt found."
                        pip install -r requirements.txt
                    else
                        echo "No requirements.txt found."
                        echo "Installing Flask and pytest..."
                        pip install Flask pytest
                    fi

                    echo ""
                    echo "Build completed successfully."
                '''
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'

                sh '''
                    set -e

                    echo "Activating virtual environment..."
                    . "$VENV/bin/activate"

                    echo ""
                    echo "Python version:"
                    python --version

                    echo ""
                    echo "Checking for tests..."

                    if [ -d tests ]; then
                        echo "tests directory found."
                        pytest -v

                    elif ls test_*.py >/dev/null 2>&1; then
                        echo "Test files found."
                        pytest -v

                    else
                        echo "No pytest test files found."
                        echo "Running Flask application syntax check..."

                        if [ -f app.py ]; then
                            python -m py_compile app.py
                            echo "app.py syntax check passed."

                        elif [ -f application.py ]; then
                            python -m py_compile application.py
                            echo "application.py syntax check passed."

                        else
                            echo "ERROR: No Flask application file found."
                            exit 1
                        fi
                    fi

                    echo ""
                    echo "Test stage completed successfully."
                '''
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying application to staging environment...'

                sh '''
                    set -e

                    . "$VENV/bin/activate"

                    echo "======================================"
                    echo "Deploying Flask Application"
                    echo "======================================"

                    # Stop an existing Flask process if present
                    if [ -f "$WORKSPACE/flask.pid" ]; then
                        echo "Stopping previous Flask application..."

                        kill "$(cat "$WORKSPACE/flask.pid")" 2>/dev/null || true

                        rm -f "$WORKSPACE/flask.pid"
                    fi

                    # Make sure app.py exists
                    if [ ! -f app.py ]; then
                        echo "ERROR: app.py not found."
                        exit 1
                    fi

                    export FLASK_APP=app.py

                    echo "Starting Flask application on port 5000..."

                    nohup flask run \
                        --host=0.0.0.0 \
                        --port=5000 \
                        > "$WORKSPACE/flask.log" 2>&1 &

                    FLASK_PID=$!

                    echo "$FLASK_PID" > "$WORKSPACE/flask.pid"

                    echo "Flask PID: $FLASK_PID"

                    echo "Waiting for application to start..."
                    sleep 5

                    echo ""
                    echo "Flask application log:"
                    cat "$WORKSPACE/flask.log" || true

                    echo ""
                    echo "Deployment completed."
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                echo 'Verifying staging deployment...'

                sh '''
                    set -e

                    echo "Waiting for application..."
                    sleep 3

                    echo "Testing http://localhost:5000..."

                    if curl -f http://localhost:5000; then
                        echo ""
                        echo "======================================"
                        echo "Deployment verification SUCCESS"
                        echo "======================================"
                    else
                        echo ""
                        echo "======================================"
                        echo "Deployment verification FAILED"
                        echo "======================================"

                        echo ""
                        echo "Flask application logs:"
                        tail -20 "$WORKSPACE/flask.log" || true

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

            mail to: 'lambhate.ankit@gmail.com',
                 subject: "Jenkins Pipeline SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: """Hello Ankit,

The Jenkins pipeline completed successfully.

Job: ${env.JOB_NAME}
Build: #${env.BUILD_NUMBER}
Status: SUCCESS

Stages completed:
- Build
- Test
- Deploy
- Verify Deployment

The Flask application was successfully deployed to the staging environment.

Please check Jenkins for the complete build details.
"""
        }

        failure {
            echo '======================================'
            echo 'Pipeline FAILED!'
            echo '======================================'

            mail to: 'lambhate.ankit@gmail.com',
                 subject: "Jenkins Pipeline FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: """Hello Ankit,

The Jenkins pipeline has FAILED.

Job: ${env.JOB_NAME}
Build: #${env.BUILD_NUMBER}
Status: FAILURE

Please check the Jenkins console output for the exact failure.

Jenkins pipeline stages:
- Build
- Test
- Deploy
- Verify Deployment

Regards,
Jenkins
"""
        }

        always {
            echo 'Pipeline execution finished.'

            sh '''
                if [ -f "$WORKSPACE/flask.pid" ]; then
                    echo "Cleaning up Flask application..."

                    kill "$(cat "$WORKSPACE/flask.pid")" 2>/dev/null || true

                    rm -f "$WORKSPACE/flask.pid"
                fi
            '''
        }
    }
}

