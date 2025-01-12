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
        }
        */

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
    }
    post {
        always {
            junit 'test-results/junit.xml' // Archive JUnit test results
        }
    }
}