pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Docker Build') {
            steps {
                sh 'docker build -t portfolio-site .'
            }
        }
        stage('Docker Push') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login \
                        -u "$DOCKER_USER" \
                        --password-stdin

                        docker tag portfolio-site:latest \
                        rachitsahni/portfolio-site:latest

                        docker push \
                        rachitsahni/portfolio-site:latest
                    '''
                }
            }
        }
        stage('Deploy to EC2') {
            steps {
                sshagent(['ec2-ssh']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ec2-user@3.108.54.105 "
                            docker pull rachitsahni/portfolio-site:latest &&
                            docker stop portfolio-site || true &&
                            docker rm portfolio-site || true &&
                            docker run -d --name portfolio-site -p 8081:80 rachitsahni/portfolio-site:latest
                        "
                    '''
                }
            }
        }
    }
}
