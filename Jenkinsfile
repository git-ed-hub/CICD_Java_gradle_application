pipeline{
    agent { label 'Jenkins-Agent' } 
    tools {
        jdk 'Java11'
        
    }
    environment{
       VERSION = "${env.BUILD_ID}"
       APP_NAME = "gradle-webapp"
       RELEASE = "1.0.0"
       DOCKER_USER = "testsysadmin8"
       DOCKER_PASS = 'dockerhub'
       IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
       IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
        
    }
    stages{
        stage("sonar quality check"){
            
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') {
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
                    withCredentials([usernamePassword(credentialsId: 'nexus', passwordVariable: 'pass', usernameVariable: 'user')]) {
                             sh '''
                                docker build -t 192.168.52.132:8083/${IMAGE_NAME}:${VERSION} .
                                docker login -u admin -p $pass 192.168.52.132:8083 
                                docker push  192.168.52.132:8083/${IMAGE_NAME}:${VERSION}
                                docker rmi 192.168.52.132:8083/${IMAGE_NAME}:${VERSION}
                            '''  
                    }
                }
            }
        }  
        stage("pushing the helm charts to nexus"){
            steps{
                script{
                    withCredentials([usernamePassword(credentialsId: 'nexus', passwordVariable: 'pass', usernameVariable: 'user')]) {
                          dir('kubernetes/') {
                             sh '''
                                 helmversion=$( helm show chart myapp | grep version | cut -d: -f 2 | tr -d ' ')
                                 tar -czvf  myapp-${helmversion}.tgz myapp/
                                 curl -u admin:$pass http://192.168.52.132:8081/repository/helm-hosted/ --upload-file myapp-${helmversion}.tgz -v
                            '''
                          }
                    }
                }
            }
        }         
    
    }
    post {
            always {
                mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "isma.19.kor@gmail.com";  
            }
    }
}
