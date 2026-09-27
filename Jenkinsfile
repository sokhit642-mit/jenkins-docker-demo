pipeline {

    agent any

    stages {

        stage('Checkout') {

            steps {

                echo 'Checking out source code...'

                checkout scm

                sh '''
                    echo "Git commit:"
                    git rev-parse HEAD

                    echo "Git branch:"
                    git branch --show-current

                    echo "Git tag:"
                    git describe --tags --exact-match || true
                '''
            }
        }


        stage('Docker Build') {

            steps {

                echo 'Building Docker image...'

                sh '''
                    docker build \
                        -t jenkins-docker-demo:test \
                        .
                '''
            }
        }


        stage('Docker Test') {

            steps {

                echo 'Checking Docker image...'

                sh '''
                    docker images jenkins-docker-demo
                '''
            }
        }
    }


    post {

        success {

            echo 'Pipeline completed successfully!'
        }

        failure {

            echo 'Pipeline failed!'
        }
    }
}