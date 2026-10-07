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
        APP_PORT = '5000'
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

    }

    post {

        success {

            archiveArtifacts artifacts: 'flask-app-*.tar.gz',
                             fingerprint: true

            echo "Build and test completed successfully."
        }

        failure {

            echo "Pipeline failed. Check the stage above for the error."
        }
    }
}
