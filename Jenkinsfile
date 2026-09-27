pipeline {

    agent any

    environment {

        // Registry & Container Configuration
        GITHUB_REPOSITORY = "https://github.com/sokhit642-mit/jenkins-docker-demo.git"
        DOCKER_REGISTRY = "ksk6699"
        DOCKER_IMAGE_NAME = "jenkins-docker-demo"

        // EC2 Instance Configuration
        AWS_EC2_HOST = "54.66.234.101"
    }

    parameters {

        gitParameter(
            name: 'TAG',
            type: 'PT_TAG',
            defaultValue: '',
            description: 'Select the Git tag to build.'
        )

        gitParameter(
            name: 'BRANCH',
            type: 'PT_BRANCH',
            defaultValue: '',
            description: 'Select the Git branch to build.'
        )

        choice(
            name: 'ACTION',
            choices: ['build', 'deploy', 'rollback'],
            description: 'Choose whether to deploy a new version or rollback to a previous version.'
        )
    }

    stages {

        stage('Checkout Code') {

            steps {

                script {

                    echo "Action: ${params.ACTION} | Branch: ${params.BRANCH} | Revision: ${params.VERSION_COMMIT}"

                    if (params.TAG) {

                        echo "Checking out tag: ${params.TAG}"

                        checkout([
                            $class: 'GitSCM',
                            branches: [[
                                name: "refs/tags/${params.TAG}"
                            ]],
                            userRemoteConfigs: [[
                                url: env.GITHUB_REPOSITORY
                            ]]
                        ])

                    } else {

                        echo "Checking out branch: ${params.BRANCH}"

                        checkout([
                            $class: 'GitSCM',
                            branches: [[
                                name: "${params.BRANCH}"
                            ]],
                            userRemoteConfigs: [[
                                url: env.GITHUB_REPOSITORY
                            ]]
                        ])
                    }
                }
            }
        }

        stage('Build') {

            when {
                expression {
                    params.ACTION == 'build'
                }
            }

            steps {

                script {

                    sh """
                        echo "Building the application..."

                        set -e

                        docker build \
                            -t ${env.DOCKER_REGISTRY}/${env.DOCKER_IMAGE_NAME}:${params.TAG} \
                            .
                    """
                }
            }
        }

        stage('Push Image') {

            when {
                expression {
                    params.ACTION == 'build'
                }
            }

            steps {

                script {

                    withCredentials([
                        usernamePassword(
                            credentialsId: 'docker-hub-id',
                            usernameVariable: 'DOCKER_USERNAME',
                            passwordVariable: 'DOCKER_PASSWORD'
                        )
                    ]) {

                        sh '''
                            echo "$DOCKER_PASSWORD" | docker login \
                                -u "$DOCKER_USERNAME" \
                                --password-stdin
                        '''
                    }

                    sh """
                        docker push \
                            ${env.DOCKER_REGISTRY}/${env.DOCKER_IMAGE_NAME}:${params.TAG}
                    """
                }
            }
        }

        stage('Deploy') {

            when {
                expression {
                    params.ACTION == 'deploy'
                }
            }

            steps {

                script {

                    echo "Deploying the application..."

                    sh """
                        ssh root@${env.AWS_EC2_HOST} \
                        "docker pull ${env.DOCKER_REGISTRY}/${env.DOCKER_IMAGE_NAME}:${params.TAG} && \
                         docker stop ${env.DOCKER_IMAGE_NAME} || true && \
                         docker rm ${env.DOCKER_IMAGE_NAME} || true && \
                         docker run -d \
                         --name ${env.DOCKER_IMAGE_NAME} \
                         -p 80:80 \
                         ${env.DOCKER_REGISTRY}/${env.DOCKER_IMAGE_NAME}:${params.TAG}"
                    """
                }
            }
        }

        stage('Rollback') {

            when {
                expression {
                    params.ACTION == 'rollback'
                }
            }

            steps {

                script {

                    echo "Rolling back the application..."

                    sh """
                        ssh root@${env.AWS_EC2_HOST} \
                        "docker stop ${env.DOCKER_IMAGE_NAME} || true && \
                         docker rm ${env.DOCKER_IMAGE_NAME} || true && \
                         docker run -d \
                         --name ${env.DOCKER_IMAGE_NAME} \
                         -p 80:80 \
                         ${env.DOCKER_REGISTRY}/${env.DOCKER_IMAGE_NAME}:${params.TAG}"
                    """
                }
            }
        }
    }
}
