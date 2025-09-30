pipeline {
    agent { label "agent1" }

    environment {
        IMAGE_NAME = '10ajaykumar/3d-boxes'
        MAIN_TAG = 'latest'
        IMAGE_TAG = '${env.BUILD_NUMBER}'
    }

    stages {
        stage('Cloning') {
            steps {
                echo "Git cloning started"
                git branch: "${BRANCH_NAME}", credentialsId: 'git_token', url: 'https://github.com/10ajaykumar/3d-boxes-background.git'
                echo "Git cloning completed"
            }
        }

        stage("Build Both Docker Images") {
            steps {
                script {
                    echo "Building Dockerfile.main → ${IMAGE_NAME}:${MAIN_TAG}"
                    sh "docker build -t ${IMAGE_NAME}:${MAIN_TAG} ."

                    echo "Building Dockerfile.image → ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                }
            }
        }

        stage("Push Docker Images") {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: "docker_token",
                        usernameVariable: "docker_tokenUser", 
                        passwordVariable: "docker_tokenPass"
                    )]) {
                        sh 'echo "$docker_tokenPass" | docker login -u "$docker_tokenUser" --password-stdin'

                        sh "docker tag ${IMAGE_NAME}:${MAIN_TAG} ${docker_tokenUser}/${IMAGE_NAME}:${MAIN_TAG}"
                        sh "docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${docker_tokenUser}/${IMAGE_NAME}:${IMAGE_TAG}"

                        sh "docker push ${docker_tokenUser}/${IMAGE_NAME}:${MAIN_TAG}"
                        sh "docker push ${docker_tokenUser}/${IMAGE_NAME}:${IMAGE_TAG}"

                        sh 'rm -f /home/aj/.docker/config.json'
                    }
                }
            }
        }


        stage("Deploy Docker Containers") {
            steps {
                script {
                    // Optional: stop old containers
                    sh "docker rm -f 3d-boxes-main || true"
                    sh "docker rm -f 3d-boxes-image || true"

                    echo "Running container from ${IMAGE_NAME}:${MAIN_TAG} on port 2121"
                    sh "docker run -d -p 2121:80 --name 3d-boxes-main ${IMAGE_NAME}:${MAIN_TAG}"

                    echo "Running container from ${IMAGE_NAME}:${IMAGE_TAG} on port 2122"
                    sh "docker run -d -p 2122:80 --name 3d-boxes-image ${IMAGE_NAME}:${IMAGE_TAG}"
                }
            }
        }
    }


    post {
        success {
            script {
                emailext (
                    to: 'ajaykumar.b@kaveriseeds.in', 
                    subject: "✅ Pipeline Successful: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                    body: """<html>
                        <body>
                            <p>✅ Pipeline succeeded on branch: <b>${env.BRANCH_NAME}</b></p>
                            <ul>
                                <li>Main image: ${IMAGE_NAME}:${MAIN_TAG}</li>
                                <li>Image version: ${IMAGE_NAME}:${IMAGE_TAG}</li>
                            </ul>
                            <p>Build number: ${env.BUILD_NUMBER}</p>
                        </body>
                    </html>""",
                    mimeType: 'text/html'
                )
            }
        }

        failure {
            script {
                emailext (
                    to: 'ajaykumar.b@kaveriseeds.in', 
                    subject: "❌ Pipeline Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                    body: """<html>
                        <body>
                            <p>❌ Pipeline failed on branch: <b>${env.BRANCH_NAME}</b></p>
                            <p>Please check Jenkins logs for details.</p>
                            <p>Build number: ${env.BUILD_NUMBER}</p>
                        </body>
                    </html>""",
                    mimeType: 'text/html'
                )
            }
        }
    }
}
