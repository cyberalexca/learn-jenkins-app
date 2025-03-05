pipeline {
    agent any

    stages {
        stage('w/o Docker') {
            steps {
                echo "Without Docker"
                // sh 'npm --version'
                sh '''
                    ls -la
                    touch no-container.txt
                '''
            }
        }
        stage('w Docker') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                echo "With Docker"
                sh 'npm --version'
                sh '''
                    ls -la
                    touch container.txt
                '''
            }
        }

    }
}
