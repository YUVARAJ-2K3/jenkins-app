pipeline {
    agent any

    stages {
        stage ('build'){
                steps {
                sh 'echo "witoutdocker"'
                sh 'npm --version'
                
            }
        }
        stage ('test'){
            steps{
                sh '''
                echo 'testing'
                echo 'tested'
                '''
            }
        } 
    }
}
