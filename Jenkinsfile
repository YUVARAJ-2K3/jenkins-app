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
            environment {
                HOME = "${WORKSPACE}"
                npm_config_cache = "${WORKSPACE}/.npm"
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
                npm install netlify-cli
                node_modules/.bin/netflix --version
                '''
            }
        }
    }
}