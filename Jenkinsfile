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
                    set -euo pipefail

                    echo "Testing virtual environment creation methods..."

                    # Method 1: Try built-in venv module
                    if python3 -m venv /tmp/test_venv_builtin >/dev/null 2>&1; then
                        echo "✓ Built-in venv module works"
                        rm -rf /tmp/test_venv_builtin
                        VENV_CMD="python3 -m venv"
                    else
                        echo "✗ Built-in venv module failed"
                    fi

                    # Method 2: Try virtualenv via pip (if not already found)
                    if [ -z "${VENV_CMD:-}" ]; then
                        echo "Trying to install virtualenv via pip..."
                        # Try different pip approaches
                        if python3 -m pip install --user virtualenv >/dev/null 2>&1; then
                            echo "✓ virtualenv installed via pip"
                            VENV_CMD="python3 -m virtualenv"
                        elif pip3 install --user virtualenv >/dev/null 2>&1; then
                            echo "✓ virtualenv installed via pip3"
                            VENV_CMD="python3 -m virtualenv"
                        else
                            echo "✗ Failed to install virtualenv via pip"
                        fi
                    fi

                    # Method 3: Check if virtualenv is already available
                    if [ -z "${VENV_CMD:-}" ]; then
                        if command -v virtualenv >/dev/null 2>&1; then
                            echo "✓ Found virtualenv command"
                            VENV_CMD="virtualenv"
                        fi
                    fi

                    # Final check
                    if [ -z "${VENV_CMD:-}" ]; then
                        echo "ERROR: No working virtual environment method found"
                        echo "Debug info:"
                        echo "  Python version: $(python3 --version)"
                        echo "  Pip version: $(python3 -m pip --version || pip3 --version || echo 'not found')"
                        echo "  User site-packages: $(python3 -m site --user-site 2>/dev/null || echo 'unknown')"
                        exit 1
                    fi

                    echo "Selected method: $VENV_CMD"
                    echo "Creating virtual environment at $VENV"

                    # Create the actual virtual environment
                    $VENV_CMD "$VENV"

                    # Activate and verify
                    . "$VENV/bin/activate"
                    echo "Virtual environment activated:"
                    python --version
                    pip --version

                    # Upgrade pip
                    pip install --upgrade pip

                    # Install requirements
                    if [ -f requirements.txt ]; then
                        echo "Installing dependencies from requirements.txt"
                        pip install -r requirements.txt
                    else
                        echo "requirements.txt not found, installing Flask and pytest"
                        pip install Flask pytest
                    fi
                '''
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'

                sh '''
                    set -euo pipefail
                    . "$VENV/bin/activate"

                    if [ -d tests ] || ls test_*.py >/dev/null 2>&1; then
                        echo "Running pytest..."
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
                    set -euo pipefail
                    . "$VENV/bin/activate"

                    # Stop any previous Flask process
                    if [ -f flask.pid ]; then
                        echo "Stopping previous Flask process..."
                        kill $(cat flask.pid) 2>/dev/null || true
                        rm -f flask.pid
                    fi

                    # Start Flask application
                    export FLASK_APP=app.py

                    echo "Starting Flask application..."
                    nohup flask run \
                        --host=0.0.0.0 \
                        --port=5000 \
                        > flask.log 2>&1 &

                    echo $! > flask.pid

                    sleep 5

                    echo "Flask application started (PID: $(cat flask.pid))."
                    echo "Log output:"
                    cat flask.log || true
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                echo 'Verifying deployment...'

                sh '''
                    set -euo pipefail
                    sleep 5  # Give app more time to start

                    if curl -f http://localhost:5000; then
                        echo ""
                        echo "✓ Application is running successfully!"
                    else
                        echo ""
                        echo "✗ Application failed to respond."
                        echo "Last 20 lines of flask.log:"
                        tail -20 flask.log || true
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
