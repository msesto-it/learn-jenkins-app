pipeline {
    agent any 

    environment {
        NETLIFY_SITE_ID = '1607d98c-5f48-46ed-ae1f-215def91dd4e'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token') // Use the Netlify token stored in Jenkins credentials
    }

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
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local', reportTitles: '', useWrapperFileDirectly: true])  
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

        stage('Deploy') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod
                '''
            }
            post {
                success {
                    echo "Deploy Stage successfully to site ID: $NETLIFY_SITE_ID"
                }
                failure {
                    echo 'Deploy Stage failed'
                }
            }
        }

        stage('PROD E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = 'https://effulgent-sfogliatella-960005.netlify.app'
                }
            steps {
                sh '''
                npx playwright test --reporter=html #Run Playwright tests
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E Report', reportTitles: '', useWrapperFileDirectly: true]) 
                }
                success {
                    echo 'PROD E2E tests passed'
                }
                failure {
                    echo 'PROD E2E tests failed'
                }
            }
        }
    }
}