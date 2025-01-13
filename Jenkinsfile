pipeline {
    agent any
    
    /*options {
        disableConcurrentBuilds() // Prevent concurrent builds
    }*/

    stages {
        /*stage('Clean Workspace') {
            steps {
                deleteDir() // Delete the workspace before starting the build
            }
        }*/
        
        /*stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    ls -la
                    node --version
                    npm --version
                    npm ci #Install dependencies
                    npm run build
                    ls -la #List files in the current directory
                '''
            }
        }*/

        stage ('Tests') {
            parallel {
                stage('Unit Tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                        test -f build/index.html
                        npm test
                        '''
                    }
                    post {
                        always {
                            junit 'jest-results/junit.xml' // Archive JUnit test results
                        }
                        success {
                            echo 'Unit tests passed'
                        }
                        failure {
                            echo 'Unit tests failed'
                        }
                    }
                }
                
                stage('E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                        npm install serve
                        node_modules/.bin/serve -s build & #Start the server in the background
                        sleep 10 #Wait for the server to start
                        npx playwright test --reporter=html #Run Playwright tests
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])  
                        }
                        success {
                            echo 'E2E tests passed'
                        }
                        failure {
                            echo 'E2E tests failed'
                        }
                    }
                }
            }
        }  
    }
}