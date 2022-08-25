pipeline{
    agent {
        label 'build-node'
    }
    stages{
        stage("sonar quality check"){
            agent {
                docker {
                    image 'openjdk:11'
                }
            }
            steps{
                script{
                    sh 'pwd'

                }  
            }
        }

    }
}
