pipeline {
    agent any

    environment {
        // GIT_HASH = GIT_COMMIT.take(7)
        GIT_HASH = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim().take(7)
        NETLIFY_SITE_ID = "fe19abea-7574-467e-a119-ab352269de48"
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:22-alpine'
                    reuseNode true
                }
            }
            steps {
                sh "echo 'REACT_APP_GIT_HASH=${GIT_HASH}' > .env"
                sh '''
                    npm ci
                    npm run build
                '''
            }
        }

        stage("Run tests") {
            parallel {
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
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
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
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: true, icon: '', keepAll: true, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }

        stage('Deploy staging') {
            agent {
                docker {
                    image 'node:22-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir build
                '''
            }
        }

        stage('Deploy production') {
            agent {
                docker {
                    image 'node:22-alpine'
                    reuseNode true
                }
            }
            steps {
                timeout(time: 30, unit: 'SECONDS') {
                    input cancel: 'No, please', message: 'Deploy to production?', ok: 'Yes, sure'
                }
                sh '''
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --prod --dir build
                '''
            }
        }
    }

    // post {
    //     always {
    //         junit 'jest-results/junit.xml'
    //         // archiveArtifacts artifacts: 'build/'
    //         publishHTML([allowMissing: false, alwaysLinkToLastBuild: true, icon: '', keepAll: true, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: '', useWrapperFileDirectly: true])
    //     }
    // }
}
