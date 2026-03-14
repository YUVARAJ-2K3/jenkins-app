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
        stage ('deploy'){
            steps{
                sh '''
                echo 'deploy'
                '''
            }
        } 
    }
}
