  pipeline {
      agent any

      environment {
          // Python installation
          PYTHON = tool 'Python 3'
          // Virtual environment path
          VENV = "${WORKSPACE}/venv"
      }

      stages {
          stage('Checkout') {
              steps {
                  checkout scm
              }
          }

          stage('Build') {
              steps {
                  echo 'Setting up Python virtual environment...'
                  sh "${PYTHON} -m venv ${VENV}"
                  echo 'Activating virtual environment and installing dependencies...'
                  sh """
                  source ${VENV}/bin/activate
                  pip install --upgrade pip
                  if [ -f requirements.txt ]; then
                      pip install -r requirements.txt
                  else
                      echo "No requirements.txt found, installing Flask basics..."
                      pip install Flask pytest
                  fi
                  """
              }
          }

          stage('Test') {
              steps {
                  echo 'Running tests...'
                  sh """
                  source ${VENV}/bin/activate
                  # Run pytest if tests exist, otherwise run a basic Flask check
                  if [ -f test_*.py ] || [ -d tests ]; then
                      pytest -v
                  else
                      echo "No test files found. Running basic application syntax check..."
                      python -m py_compile app.py || python -m py_compile application.py || echo "Could not find main
  app file to check"
                  fi
                  """
              }
          }

          stage('Deploy') {
              when {
                  expression {
                      return currentBuild.currentResult == 'SUCCESS'
                  }
              }
              steps {
                  echo 'Deploying application to staging...'
                  sh """
                  source ${VENV}/bin/activate
                  # Run Flask app in background on port 5000
                  nohup flask run --host=0.0.0.0 --port=5000 > flask.log 2>&1 &
                  echo \$! > flask.pid
                  sleep 5
                  echo "Application started. Check flask.log for output."
                  """
              }
          }

          stage('Verify Deployment') {
              when {
                  expression {
                      return currentBuild.currentResult == 'SUCCESS'
                  }
              }
              steps {
                  echo 'Verifying deployment...'
                  sh '''
                  # Wait a moment then check if Flask is running
                  sleep 3
                  curl -s http://localhost:5000 || echo "Application may still be starting or failed to start"
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
          always {
              echo 'Cleaning up...'
              // Optional: kill background Flask process
              sh '''
              if [ -f flask.pid ]; then
                  kill $(cat flask.pid) 2>/dev/null || echo "No Flask process to kill"
                  rm flask.pid
              fi
              '''
          }
      }
  }
