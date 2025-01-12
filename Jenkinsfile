pipeline {
    agent any

    stages {
        /*
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
        } */

        stage('Tests') {
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
        }
        
        stage('E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.49.1-jammy'
                    reuseNode true
                }
            }
            steps {
                sh '''
                npm install serve #Install serve Locally
                workspaces/learn-jenkins-app/node_modules/serve -s build
                npx playwright test
                '''
            }
        }
        
    }
    post {
        always {
            junit 'test-results/junit.xml' // Archive JUnit test results
        }
    }
}