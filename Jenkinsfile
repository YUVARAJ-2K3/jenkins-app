pipeline {
    agent any

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
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }
        stage('Test'){
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                echo 'Test Stage'
                sh 'test -f build/index.html'
                sh 'npm test'
            }
        }
        stage('Deploy'){
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                echo 'Deploy Stage'
                sh '''
                apk add --no-cache python3 make g++
                npm install --no-save netlify-cli
                node_modules/.bin/netlify --version
                '''
                echo 'Deploy Success'
            }
        }
    }
}