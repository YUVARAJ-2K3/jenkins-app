pipeline {
    agent any

    stages {
        stage ('docker'){
                steps {
                sh 'echo "witoutdocker"'
                sh 'npm --version'
                sh 'ls'
                sh 'cd jenkins-app'
                sh 'npm install'
                sh 'npm run start'
                
            }
        } 
    }
}
