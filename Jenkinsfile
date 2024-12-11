pipeline {
    agent any
 
    stages {
        stage('with Docker') {
            agent{
                docker{
                    image 'node:18-alpine'
                }
            }
            steps {
                sh '''
                    echo "with docker"
                    npm --version
                    touch "with-container.txt"
                '''
            }
        }
        stage('without Docker') {
            steps {
                sh '''
                    echo "without docker"
                    touch "without-container.txt"
                    touch "test.text"
                '''
            }
        }
       
    }
}