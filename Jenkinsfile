pipeline {
agent any

environment {
    PYTHON_PACKAGES = "${WORKSPACE}/python_packages"
    PATH = "${WORKSPACE}/python_packages/bin:${env.PATH}"
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

            sh """
                set -e

                echo "Python version:"
                python3 --version

                echo "Python location:"
                which python3

                echo "Cleaning previous build..."
                rm -rf "${PYTHON_PACKAGES}"
                mkdir -p "${PYTHON_PACKAGES}"

                echo "Installing dependencies..."

                if [ -f requirements.txt ]; then
                    python3 -m pip install --target="${PYTHON_PACKAGES}" -r requirements.txt
                else
                    echo "ERROR: requirements.txt not found"
                    exit 1
                fi

                echo "Verifying Flask installation..."

                PYTHONPATH="${PYTHON_PACKAGES}" python3 -c "import flask; print('Flask version:', flask.__version__)"

                echo "Build completed successfully."
            """
        }
    }

    stage('Test') {
        steps {
            echo 'Running tests...'

            sh """
                set -e

                export PYTHONPATH="${PYTHON_PACKAGES}"

                echo "Python version:"
                python3 --version

                echo "Checking for project tests..."

                if [ -d tests ]; then
                    echo "tests/ directory found."
                    python3 -m pytest tests/ -v

                elif [ -f test_app.py ]; then
                    echo "test_app.py found."
                    python3 -m pytest test_app.py -v

                elif ls test_*.py >/dev/null 2>&1; then
                    echo "Project test files found."
                    python3 -m pytest test_*.py -v

                else
                    echo "No pytest tests found."
                    echo "Running Python syntax check..."

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

                echo "Tests completed successfully."
            """
        }
    }

    stage('Deploy') {
        steps {
            echo 'Deploying Flask application...'

            sh """
                set -e

                export PYTHONPATH="${PYTHON_PACKAGES}"

                if [ -f flask.pid ]; then
                    OLD_PID=\$(cat flask.pid)

                    if kill -0 "\$OLD_PID" 2>/dev/null; then
                        echo "Stopping existing Flask process: \$OLD_PID"
                        kill "\$OLD_PID" || true
                        sleep 2
                    fi

                    rm -f flask.pid
                fi

                echo "Starting Flask application..."

                export FLASK_APP=app.py

                nohup python3 -m flask run --host=0.0.0.0 --port=5000 > flask.log 2>&1 &

                FLASK_PID=\$!
                echo "\$FLASK_PID" > flask.pid

                echo "Flask PID: \$FLASK_PID"

                sleep 5

                if kill -0 "\$FLASK_PID" 2>/dev/null; then
                    echo "Flask process is running."
                else
                    echo "ERROR: Flask process stopped."
                    cat flask.log
                    exit 1
                fi

                echo "Flask application started successfully."
            """
        }
    }

    stage('Verify Deployment') {
        steps {
            echo 'Verifying deployment...'

            sh """
                set -e

                sleep 3

                echo "Checking http://localhost:5000 ..."

                if curl -f --max-time 10 http://localhost:5000; then
                    echo "Application is running successfully!"
                else
                    echo "Application verification failed."
                    echo "Last 30 lines of flask.log:"
                    tail -30 flask.log || true
                    exit 1
                fi
            """
        }
    }
}

post {

    success {
        echo 'Pipeline completed successfully!'

        mail(
            to: 'lambhate.ankit@gmail.com',
            subject: "Jenkins Pipeline SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            body: "The Flask application was built, tested, deployed and verified successfully."
        )
    }

    failure {
        echo 'Pipeline failed!'

        mail(
            to: 'lambhate.ankit@gmail.com',
            subject: "Jenkins Pipeline FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            body: "The Jenkins pipeline failed. Please check the Jenkins console output."
        )
    }

    always {
        echo 'Pipeline execution finished.'

        sh """
            if [ -f flask.pid ]; then
                PID=\$(cat flask.pid)

                if kill -0 "\$PID" 2>/dev/null; then
                    echo "Flask process is still running: \$PID"
                else
                    echo "Flask process is not running."
                fi
            fi
        """
    }
}
}
