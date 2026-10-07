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
                        scp -o StrictHostKeyChecking=no \
                            flask-app-${params.APP_VERSION}.tar.gz \
                            ec2-user@${SERVER_IP}:${DEPLOY_DIR}/

                        ssh -o StrictHostKeyChecking=no \
                            ec2-user@${SERVER_IP} '
                                cd ${DEPLOY_DIR}

                                tar -xzf flask-app-${params.APP_VERSION}.tar.gz

                                python3 -m venv venv

                                . venv/bin/activate

                                pip install -r requirements.txt

                                pkill -f "gunicorn.*:${APP_PORT}" || true

                                nohup gunicorn \
                                    --bind 0.0.0.0:${APP_PORT} \
                                    --workers 2 \
                                    app:app \
                                    > gunicorn.log 2>&1 &

                                echo "Application deployed successfully"
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

                echo "Health check passed."
            }
        }
    }

    post {

        success {

            archiveArtifacts artifacts: 'flask-app-*.tar.gz',
                             fingerprint: true

            echo "======================================"
            echo "Deployment Successful!"
            echo "Application: ${APP_NAME}"
            echo "Version: ${params.APP_VERSION}"
            echo "Environment: ${params.DEPLOY_ENV}"
            echo "Port: ${APP_PORT}"
            echo "======================================"
        }

        failure {

            echo "Pipeline failed. Check the stage above."
        }
    }
}
