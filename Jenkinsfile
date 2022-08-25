pipeline {
    agent {
        label 'kube-slave'
    }
    stages {
        stage('Build') {
            agent {
                label 'kube-slave'
            }
            steps {
                sh 'sudo apt install git'
            }
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
}