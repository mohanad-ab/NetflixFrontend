pipeline {
    agent {
        docker {
            image 'docker:19.03.12' // Specify the Docker image with Docker installed
            args '-v /var/run/docker.sock:/var/run/docker.sock' // Mount Docker socket
        }
    }

    triggers {
        githubPush() // Trigger the pipeline upon push event in GitHub
    }

    options {
        timeout(time: 10, unit: 'MINUTES') // Discard the build after 10 minutes of running
        timestamps() // Display timestamp in console output
    }

    stages {
        stage('Prepare Environment Variables') {
            steps {
                script {
                    env.GIT_COMMIT = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                    env.TIMESTAMP = sh(script: 'date +"%Y%m%d-%H%M%S"', returnStdout: true).trim()
                    env.IMAGE_TAG = "${env.GIT_COMMIT}-${env.TIMESTAMP}"
                    env.IMAGE_BASE_NAME = 'netflix-app'
                }
            }
        }

        stage('Docker setup') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    script {
                        sh 'docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD'
                    }
                }
            }
        }

        stage('Build & Push') {
            steps {
                script {
                    def imageName = "${env.DOCKER_USERNAME}/${env.IMAGE_BASE_NAME}:${env.IMAGE_TAG}"
                    sh "docker build -t ${imageName} ."
                    sh "docker push ${imageName}"
                }
            }
        }
    }
}
