pipeline {
    agent any
    stage("docker build & docker push"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'docker_secret', variable: 'docker_secret')]) {
                             sh '''
                             docker build -t bill3213/springapp:1 .
                             docker login -u bill3213 -p $docker_secret
                             docker push bill3213/springapp:1
                             docker rmi bill3213/springapp:1
                             '''
                    }
                }
            }

    }
}