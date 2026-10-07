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

    stages {

        stage('Build') {
    steps {

        echo "Building Flask application version ${params.APP_VERSION}"

        sh '''
            tar -czf flask-app-${APP_VERSION}.tar.gz \
                app.py \
                requirements.txt \
                templates
        '''

        echo 'Build completed successfully.'
    }
}

        stage('Test') {
            steps {

                echo 'Testing Flask application...'

                sh '''
                    python3 -c "from app import app; print('Flask application test passed')"
                '''
            }
        }

    }

    post {

        success {

            archiveArtifacts artifacts: 'flask-app-*.tar.gz',
                             fingerprint: true
        }
    }
}
