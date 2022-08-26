pipeline {
    agent any
    stages{
        stage("sonar quality check"){
            agent {
                label 'kube-slave'
            }
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
            agent {
                label 'azure'
            }
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
        stage('indentifying misconfigs using datree in helm charts'){
            steps{
                script{

                    dir('kubernetes/') {
                           sh 'helm datree test myapp/ --no-record'
                    }
                }
            }
        }
        stage("pushing the helm charts to nexus"){
            steps{
                dir('kubernetes/') {
                             sh '''
                                 helmversion=$( helm show chart myapp | grep version | cut -d: -f 2 | tr -d ' ')
                                 tar -czvf  myapp-${helmversion}.tgz myapp/
                                 aws ecr get-login-password --region us-west-2 | helm registry login | --username AWS | --password-stdin 487936429785.dkr.ecr.region.amazonaws.com
                                 helm push myapp-${helmversion}.tgz oci://487936429785.dkr.ecr.region.amazonaws.com/
                            '''
                }
            }
        }
        stage('Deploying application on k8s cluster') {
            steps {
               script{
                   withCredentials([kubeconfigFile(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                        dir('kubernetes/') {
                          sh 'helm upgrade --install --set image.repository="487936429785.dkr.ecr.region.amazonaws.com/myapp" --set image.tag="${VERSION}" myjavaapp myapp/ '
                        }
                    }
               }
            }
        }
    }
    post {
		always {
			mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "markwin1999@gmail.com";
		}
	}
}
