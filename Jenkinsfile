pipeline {
    agent {
        label 'azure'
    }
    stages{
        stage("sonar quality check"){
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'sonar-token') {
                            sh 'chmod +x gradlew'
                            sh './gradlew sonarqube'
                    }

                    timeout(time: 1, unit: 'HOURS') {
                      def qg = waitForQualityGate()
                      if (qg.status != 'OK') {
                           error "Pipeline aborted due to quality gate failure: ${qg.status}"
                      }
                    }

                }
            }
        }

        stage("docker build & docker push"){
                steps{
                    script{
                        withCredentials([string(credentialsId: 'docker_secret', variable: 'docker_secret')]) {
                             sh '''
                             docker build -t bill3213/springapp:$BUILD_NUMBER .
                             docker login -u bill3213 -p $docker_secret
                             docker push bill3213/springapp:$BUILD_NUMBER
                             docker rmi bill3213/springapp:$BUILD_NUMBER
                             '''
                        }
                    }
                }
        }
    }
}
