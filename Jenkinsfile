pipeline {
    agent any

    stages {
        stage ('build'){
                steps {
                sh 'echo "witoutdocker"'
                sh 'npm --version'
                sh 'ls'
                sh 'npm install'
                sh 'npm run build'
                sh 'echo "build done"'
            }
        }
        stage ('test'){
            steps{
                sh '''
                echo 'testing'
                npm test
                echo 'tested'
                '''
            }
        } 
        stage ('deploy'){
            steps{
                sh '''
                echo 'deploying'
                npm install -g serve
                serve -s build
                '''
            }
        }
    }
}
