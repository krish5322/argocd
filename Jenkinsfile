pipeline {
    agent any
    environment{
        VERSION = "${env.BUILD_ID}"
    }
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
                             docker build -t bill3213/springapp:${VERSION} .
                             docker login -u bill3213 -p $docker_secret
                             docker push bill3213/springapp:${VERSION}
                             docker rmi bill3213/springapp:${VERSION}
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
        stage("pushing the helm charts to ECR"){
            steps{
                withCredentials([string(credentialsId: 'sonar-token', variable: 'nexus_repo')]) {
                }
                dir('kubernetes/') {
                             sh '''
                                 helmversion=$( helm show chart myapp | grep version | cut -d: -f 2 | tr -d ' ')
                                 tar -czvf  myapp-${helmversion}.tgz myapp/
                                 curl -u admin:$nexus_repo http://20.39.224.51:8081/repository/helm-repo/ --upload-file myapp-${helmversion}.tgz -v
                             '''
                }
            }
        }
        stage('Deploying application on k8s cluster') {
            steps {
               script{
                   withCredentials([kubeconfigFile(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                        dir('kubernetes/') {
                          sh 'helm upgrade --install --set image.repository="bill3213/springapp" --set image.tag="${VERSION}" myjavaapp myapp/ '
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
