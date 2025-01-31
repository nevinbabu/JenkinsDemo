pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'discoverdevops/my-app:latest'  // DockerHub repo
        GIT_REPO = 'https://github.com/discover-devops/JenkinsDemo.git'  // GitHub repo
    }

    stages {
        stage('Clone Source Code') {
            steps {
                git branch: 'main', url: "${GIT_REPO}"
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image using the provided Dockerfile
                    def appImage = docker.build(DOCKER_IMAGE)
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    // Push the Docker image to DockerHub using stored credentials
                    docker.withRegistry('', 'dockerhub-credentials') {
                        docker.image(DOCKER_IMAGE).push()
                    }
                }
            }
        }

        stage('Deploy Application') {
            steps {
                script {
                    // Stop and remove any existing container with the same name, then deploy the new container
                    sh """
                    docker stop my-app || true
                    docker rm my-app || true
                    docker run -d --name my-app -p 5000:5000 ${DOCKER_IMAGE}
                    """
                }
            }
        }
    }

    post {
        always {
            echo 'Cleaning up workspace...'
            cleanWs()
        }
    }
}
