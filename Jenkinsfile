pipeline {
    agent any
    
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout code from Git repository
                checkout scm
            }
        }

        stage('Set Up Python and Install Dependencies') {
            steps {
                script {
                    if (isUnix()) {
                        sh 'python3 -m venv venv'
                        sh 'source venv/bin/activate && pip install -r requirements.txt'
                    } else {
                        bat 'python -m venv venv'
                        bat '.\\venv\\Scripts\\activate && pip install -r requirements.txt'
                    }
                }
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    if (isUnix()) {
                        sh 'source venv/bin/activate && python -m unittest discover -s tests'
                    } else {
                        bat '.\\venv\\Scripts\\activate && python -m unittest discover -s tests'
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t flask-app .'
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('', DOCKERHUB_CREDENTIALS) {
                        sh 'docker tag flask-app your-dockerhub-username/flask-app'
                        sh 'docker push your-dockerhub-username/flask-app'
                    }
                }
            }
        }

        stage('Deploy to Server') {
            steps {
                sshagent (credentials: ['ssh-key-credentials']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no user@your-server-ip << EOF
                            docker pull your-dockerhub-username/flask-app
                            docker-compose -f /path/to/docker-compose.yml up -d
                        EOF
                    '''
                }
            }
        }
    }
    
    post {
        always {
            echo 'Cleaning up workspace'
            cleanWs()
        }
    }
}
