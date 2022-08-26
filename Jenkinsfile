pipeline {
    agent {
        label 'kube-slave'
    }
    stages {
        stage('Build') {
            steps {
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
    }
    stage("docker build & docker push"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'docker_secret', variable: 'docker_secret')]) {
                             sh '''
                                docker build -t bill3213/springapp:1 .
                                docker login -u bill3213 -p $docker_secret 34.125.214.226:8083
                                docker push  bill3213/springapp:1
                                docker rmi bill3213/springapp:1
                            '''
                    }
                }
            }
        }
}