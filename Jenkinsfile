pipeline {
    agent any
    environment {
    DOCKERHUB_CREDENTIALS = credentials('docker-token')
    }
    stages {
        stage("docker build & docker push"){
            agent {
                label 'azure'
            }
            steps{
                sh '''
                   docker build -t bill3213/springapp:1 .
                   sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                   docker push  bill3213/springapp:1
                   docker rmi bill3213/springapp:1
                '''

            }
        }
    }

}