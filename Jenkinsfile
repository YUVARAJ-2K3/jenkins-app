pipeline {
    agent any

    environment {
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
                echo 'Build Stage'

                sh '''
                    node --version
                    npm --version

                    npm ci
                    npm run build

                    test -f build/index.html
                '''
            }
        }


        stage('Tests') {
            parallel {

                stage('Unit Tests') {
                    agent {
                        docker {
                            image 'node:18-bookworm'
                            reuseNode true
                        }
                    }

                    steps {
                        echo 'Unit Test Stage'

                        sh '''
                            npm test
                        '''
                    }

                    post {
                        always {
                            junit allowEmptyResults: true,
                                  testResults: 'jest-results/junit.xml'
                        }
                    }
                }


                stage('Local E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }

                    steps {
                        echo 'Local E2E Stage'

                        sh '''
                            npm install --no-save serve

                            node_modules/.bin/serve -s build > /tmp/serve.log 2>&1 &
                            SERVER_PID=$!

                            sleep 10

                            npx playwright test --reporter=html
                            TEST_EXIT=$?

                            kill $SERVER_PID || true

                            exit $TEST_EXIT
                        '''
                    }

                    post {
                        always {
                            publishHTML([
                                allowMissing: true,
                                alwaysLinkToLastBuild: false,
                                keepAll: true,
                                reportDir: 'playwright-report',
                                reportFiles: 'index.html',
                                reportName: 'Local E2E'
                            ])
                        }
                    }
                }
            }
        }


        stage('Deploy Staging') {
            agent {
                docker {
                    image 'node:18-bookworm'
                    reuseNode true
                }
            }

            steps {
                echo 'Deploying to Netlify staging'

                sh '''
                    npm install --no-save netlify-cli 

                    node_modules/.bin/netlify --version

                    node_modules/.bin/netlify deploy \
                        --site "$NETLIFY_SITE_ID" \
                        --auth "$NETLIFY_ACCESS_TOKEN" \
                        --dir=build \
                        --json > deploy-output.json

                    cat deploy-output.json
                '''

                script {
                    env.STAGING_URL = sh(
                        script: '''
                            node -e "const d=require('./deploy-output.json'); console.log(d.deploy_url || d.deployUrl || d.url)"
                        ''',
                        returnStdout: true
                    ).trim()

                    echo "Staging deployment created"
                    echo "Staging URL: ${env.STAGING_URL}"
                }
            }
        }


        stage('Staging E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = "${env.STAGING_URL}"
            }

            steps {
                echo "Running E2E against staging"

                sh '''
                    echo "Testing: $CI_ENVIRONMENT_URL"
                    npx playwright test --reporter=html
                '''
            }

            post {
                always {
                    publishHTML([
                        allowMissing: true,
                        alwaysLinkToLastBuild: false,
                        keepAll: true,
                        reportDir: 'playwright-report',
                        reportFiles: 'index.html',
                        reportName: 'Staging E2E'
                    ])
                }
            }
        }


        stage('Approval') {
            steps {
                timeout(time: 15, unit: 'MINUTES') {
                    input(
                        message: 'Staging tests passed. Deploy to production?',
                        ok: 'Deploy to Production'
                    )
                }
            }
        }


        stage('Deploy Production') {
            agent {
                docker {
                    image 'node:18-bookworm'
                    reuseNode true
                }
            }

            steps {
                echo 'Deploying to Netlify production'

                sh '''
                    node_modules/.bin/netlify deploy \
                        --site "$NETLIFY_SITE_ID" \
                        --auth "$NETLIFY_ACCESS_TOKEN" \
                        --dir=build \
                        --prod \
                        --json > production-deploy.json

                    cat production-deploy.json
                '''

                script {
                    env.PRODUCTION_URL = sh(
                        script: "node -e \"console.log(require('./production-deploy.json').url || require('./production-deploy.json').deploy_url)\"",
                        returnStdout: true
                    ).trim()

                    echo "Production deployment completed"
                    echo "Production URL: ${env.PRODUCTION_URL}"
                }
            }
        }


        stage('Production E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = "${env.PRODUCTION_URL}"
            }

            steps {
                echo 'Running E2E against production'

                sh '''
                    echo "Testing: $CI_ENVIRONMENT_URL"
                    npx playwright test --reporter=html
                '''
            }

            post {
                always {
                    publishHTML([
                        allowMissing: true,
                        alwaysLinkToLastBuild: false,
                        keepAll: true,
                        reportDir: 'playwright-report',
                        reportFiles: 'index.html',
                        reportName: 'Production E2E'
                    ])
                }
            }
        }
    }


    post {

        success {
            echo 'Pipeline succeeded!'
            echo 'Build, tests and production deployment completed successfully.'
        }

        failure {
            echo 'Pipeline failed!'
        }

        aborted {
            echo 'Pipeline aborted.'
        }

        always {
            echo 'Cleaning workspace...'
            cleanWs()
        }
    }
}
