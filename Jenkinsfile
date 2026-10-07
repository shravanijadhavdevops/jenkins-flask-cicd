pipeline {

    agent any

    parameters {

        choice(
            name: 'DEPLOY_ENV',
            choices: ['staging', 'production'],
            description: 'Select deployment environment'
        )

        string(
            name: 'APP_VERSION',
            defaultValue: '1.0',
            description: 'Enter application version'
        )
    }

    environment {

        APP_NAME = 'jenkins-flask-cicd'
        APP_PORT = '5001'
        SERVER_IP = '54.208.147.23'
        DEPLOY_DIR = '/home/ec2-user/jenkins-flask-app'
    }

    stages {

        stage('Build') {

            steps {

                echo "Building ${APP_NAME} version ${params.APP_VERSION}"

                sh """
                    tar -czf flask-app-${params.APP_VERSION}.tar.gz \
                        app.py \
                        requirements.txt \
                        templates
                """

                echo 'Build completed successfully.'
            }
        }


        stage('Test') {

            steps {

                echo "Testing Flask application..."

                sh '''
                    python3 -m venv venv

                    . venv/bin/activate

                    pip install -r requirements.txt

                    python3 -c "from app import app; print('Flask application test passed')"
                '''
            }
        }


        stage('Manual Approval') {

            steps {

                input message: "Deploy version ${params.APP_VERSION} to ${params.DEPLOY_ENV}?",
                      ok: "Approve Deployment"
            }
        }


        stage('Deploy') {

            steps {

                echo "Deploying ${APP_NAME} version ${params.APP_VERSION}..."

                sshagent(['ec2-target-key']) {

                    sh """
                        echo "Uploading application package..."

                        scp -o StrictHostKeyChecking=no \
                            flask-app-${params.APP_VERSION}.tar.gz \
                            ec2-user@${SERVER_IP}:${DEPLOY_DIR}/


                        echo "Connecting to target EC2..."

                        ssh -o StrictHostKeyChecking=no \
                            ec2-user@${SERVER_IP} '
                            
                            set -e

                            cd ${DEPLOY_DIR}


                            echo "Extracting application..."

                            tar -xzf flask-app-${params.APP_VERSION}.tar.gz


                            echo "Creating virtual environment..."

                            python3 -m venv venv


                            echo "Installing application dependencies..."

                            . venv/bin/activate

                            pip install -r requirements.txt


                            echo "Stopping previous application..."

                            if [ -f gunicorn.pid ]; then

                                kill \$(cat gunicorn.pid) || true

                                rm -f gunicorn.pid

                            fi


                            echo "Starting Gunicorn..."

                            nohup ./venv/bin/gunicorn \
                                --bind 0.0.0.0:${APP_PORT} \
                                --workers 2 \
                                app:app \
                                > gunicorn.log 2>&1 &


                            echo \$! > gunicorn.pid


                            sleep 3


                            echo "Checking Gunicorn process..."

                            if kill -0 \$(cat gunicorn.pid) 2>/dev/null; then

                                echo "Application deployed successfully"

                            else

                                echo "Application failed to start"

                                echo "========== Gunicorn Log =========="

                                cat gunicorn.log

                                echo "==================================="

                                exit 1

                            fi

                        '
                    """
                }
            }
        }


        stage('Health Check') {

            steps {

                echo "Checking application health..."

                sshagent(['ec2-target-key']) {

                    sh """
                        ssh -o StrictHostKeyChecking=no \
                            ec2-user@${SERVER_IP} \
                            "curl -f http://127.0.0.1:${APP_PORT}/health"
                    """
                }

                echo "Health check passed successfully."
            }
        }
    }


    post {

        success {

            archiveArtifacts artifacts: 'flask-app-*.tar.gz',
```
