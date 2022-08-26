pipeline {
    agent any
    stages {
        stage("docker build & docker push"){
            agent {
                label 'azure'
            }
            steps{
                script{
                    withCredentials([usernameColonPassword(credentialsId: 'docker-token', variable: 'docker_token')]) {
                             sh '''
                                docker build -t bill3213/springapp:1 .
                                docker login -u bill3213 -p $docker_token
                                docker push  bill3213/springapp:1
                                docker rmi bill3213/springapp:1
                            '''
                    }
                }
            }
        }
    }

}