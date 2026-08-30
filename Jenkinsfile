pipeline {
    agent any

    environment {
        PYTHONPATH = "${WORKSPACE}/python_packages"
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
                    echo "Creating Python package directory..."

                    rm -rf "$WORKSPACE/python_packages"
                    mkdir -p "$WORKSPACE/python_packages"

                    echo ""
                    echo "Installing dependencies..."

                    if [ -f requirements.txt ]; then
                        echo "requirements.txt found."
                        python3 -m pip install \
                            --target="$WORKSPACE/python_packages" \
                            -r requirements.txt
                    else
                        echo "requirements.txt not found."
                        echo "Installing Flask and pytest..."

                        python3 -m pip install \
                            --target="$WORKSPACE/python_packages" \
                            Flask pytest
                    fi

                    echo ""
                    echo "Installed packages:"
                    ls "$WORKSPACE/python_packages" | head -30

                    echo ""
                    echo "Testing Flask import..."

                    PYTHONPATH="$WORKSPACE/python_packages" \
                        python3 -c "import flask; print('Flask:', flask.__version__)"

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

                    export PYTHONPATH="$WORKSPACE/python_packages"

                    echo "Python version:"
                    python3 --version

                    echo ""
                    echo "Checking for tests..."

                    if [ -d tests ]; then

                        echo "tests directory found."
                        python3 -m pytest -v

                    elif ls test_*.py >/dev/null 2>&1; then

                        echo "Test files found."
                        python3 -m pytest -v

                    else

                        echo "No pytest test files found."
                        echo "Running Flask application syntax check..."

                        if [ -f app.py ]; then

                            python3 -m py_compile app.py
                            echo "app.py syntax check passed."

                        elif [ -f application.py ]; then

                            python3 -m py_compile application.py
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

                    export PYTHONPATH="$WORKSPACE/python_packages"

                    echo "======================================"
                    echo "Deploying Flask Application"
                    echo "======================================"

                    # Stop previous Flask process
                    if [ -f "$WORKSPACE/flask.pid" ]; then

                        echo "Stopping previous Flask process..."

                        kill "$(cat "$WORKSPACE/flask.pid")" 2>/dev/null || true

                        rm -f "$WORKSPACE/flask.pid"
                    fi

                    # Check application file
                    if [ ! -f app.py ]; then

                        echo "ERROR: app.py not found."
                        exit 1

                    fi

                    export FLASK_APP=app.py

                    echo "Starting Flask application..."

                    nohup python3 -m flask run \
                        --host=0.0.0.0 \
                        --port=5000 \
                        > "$WORKSPACE/flask.log" 2>&1 &

                    FLASK_PID=$!

                    echo "$FLASK_PID" > "$WORKSPACE/flask.pid"

                    echo "Flask PID: $FLASK_PID"

                    echo ""
                    echo "Waiting for Flask application..."
                    sleep 5

                    echo ""
                    echo "Flask logs:"
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

                    echo ""
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
                        tail -30 "$WORKSPACE/flask.log" || true

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

