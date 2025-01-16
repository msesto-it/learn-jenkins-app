pipeline {
    agent any 

    environment {
        NETLIFY_SITE_ID = '1607d98c-5f48-46ed-ae1f-215def91dd4e'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token') // Use the Netlify token stored in Jenkins credentials
    }

// Build docker image for Playwright
    stages {
        stage('Docker Build') {
            steps {
                sh 'docker build -t my-playwright .'
            }
        }

// Build Stage
        stage('Build') {
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
        }

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
                            junit 'test-results/junit.xml' // Archive JUnit test results
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
                            image 'my-playwright'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                        serve -s build & #Start the server in the background
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

//Merged Deploy Staging to Staging E2EDeploy to staging using Netlify
        stage('Deploy Staging') {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = "${env.STAGING_URL}"
                }
            steps {
                sh '''
                netlify --version
                netlify status
                netlify deploy --dir=build --json > deploy-output.json #If you dont put "--prod" it will deploy to staging
                CI_ENVIRONMENT_URL=$(node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json)
                npx playwright test --reporter=html #Run Playwright tests
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E Staging Report', reportTitles: '', useWrapperFileDirectly: true]) 
                }
                success {
                    echo "Staging Deploy successfull to site ID: $NETLIFY_SITE_ID"
                }
                failure {
                    echo 'Staging Deploy failed'
                }
            }
        }

// Approve stage. This will pause the pipeline and wait for user input to proceed
        stage ('Approve Deployment') {
            steps {
                input message: 'Do you wish to deploy to Production?', ok: 'Yes, I am sure!'
            }
        }

// Merged Deploy Prd with Prd E2E, Run E2E tests on the production environment
        stage('Deploy Prod') {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = 'https://effulgent-sfogliatella-960005.netlify.app'
                }
            steps {
                sh '''
                node --version #Print the Node.js version which is inbuilt in playwright image
                netlify --version
                netlify status
                netlify deploy --dir=build --prod
                npx playwright test --reporter=html #Run Playwright tests
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E Prod Report', reportTitles: '', useWrapperFileDirectly: true]) 
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