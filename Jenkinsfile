pipeline {
    agent any

    environment {
        GIT_HASH = GIT_COMMIT.take(8)
    }

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo $GIT_HASH
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }
        stage("Test") {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }

            steps {
                sh '''
                    test -f build/index.html
                    npm run test
                '''
                
            }
        }
        stage("E2E") {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.50.1-noble'
                    reuseNode true
                }
            }

            steps {
                sh '''
                    npm install serve
                    node_modules/.bin/serve -s build &
                    sleep 10
                    npx playwright test --reporter=html
                '''
                
            }
        }
    }

    post {
        always {
            junit 'jest-results/junit.xml'
            // archiveArtifacts artifacts: 'build/'
            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: '', useWrapperFileDirectly: true])
        }
    }
}
