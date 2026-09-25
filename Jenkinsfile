pipeline {
    agent any

    environment{
        NETLIFY_SITE_ID = "123456789abcdef"
    }

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-bookworm'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    node --version
                    npm --version
                    npm ci
                    npm run build
                '''
            }
        }

        stage('Test') {
            agent {
                docker {
                    image 'node:18-bookworm'
                    reuseNode true
                }
            }
            steps {
                echo 'Test Stage'
                sh 'test -f build/index.html'
                sh 'npm test'
            }
        }

        stage('Deploy') {
            agent {
                docker {
                    image 'node:18-bookworm'
                    reuseNode true
                }
            }
            steps {
                echo 'Deploy Stage'
                sh '''
                    npm install --no-save netlify-cli
                    node_modules/.bin/netlify --version
                '''
                echo 'Deploy Success'
                echo "Deploying to Netlify site ID: ${NETLIFY_SITE_ID}"
            }
        }
    }
}