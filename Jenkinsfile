pipeline {
    agent any

    stages {
      stage('Build') {
        steps {
          script {
            dockerImage = docker.build("zakir279/zakir-cv:${env.BUILD_ID}")
        }
    }
}
        stage('Push') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Test') {
            steps {
                sh 'ls -l index.html' // Simple check for index.html
            }
        }

        stage('Deploy') {
            steps {
                script {
                    // Deploy the new version
                    sshPublisher(
                        publishers: [
                            sshPublisherDesc(
                                configName: "Zakir", 
                                transfers: [sshTransfer(
                                    execCommand: """
                                        docker pull zakir279/zakir-cv:${env.BUILD_ID}
                                        docker stop zakir279/zakir-cv-container || true
                                        docker rm zakir279/zakir-cv-container || true
                                        docker run -d --name zakir279/zakir-cv -p 400:80 zakir279/zakir-cv:${env.BUILD_ID}
                                    """
                                )]
                            )
                        ]
                    )

                    
                }
            }
        }
    }

    post {
        failure {
            mail(
                to: 'zakir809288@gmail.com',
                subject: "Failed Pipeline: ${env.JOB_NAME} [${env.BUILD_NUMBER}]",
                body: "Something is wrong with the build ${env.BUILD_URL}"
            )
        }
    }
}