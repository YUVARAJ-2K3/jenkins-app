pipeline {
    agent any

    environment{
        NETLIFY_SITE_ID = credentials('netlify_site_id')
        NETLIFY_ACCESS_TOKEN = credentials('netlify_access_token')
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
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --site $NETLIFY_SITE_ID --auth $NETLIFY_ACCESS_TOKEN --prod --dir=build
                '''
            }
        }

        OnFailure {
            echo 'Build failed!'
        }

        OnSuccess {
            echo 'Build succeeded!'
        }
    }
}