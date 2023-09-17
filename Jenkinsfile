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
                    withCredentials([gitUsernamePassword(credentialsId: 'dockerhub', variable: 'docker_password'))]) {
                             sh '''
                                docker build -t 192.168.52.132:8083/${IMAGE_NAME}:${VERSION} .
                                docker login -u ${DOCKER_USER} -p $docker_password 192.168.52.132:8083 
                                docker push  192.168.52.132:8083/${IMAGE_NAME}:${VERSION}
                                docker rmi 192.168.52.132:8083/${IMAGE_NAME}:${VERSION}
                            '''  
                    }
                }
            }
        }   
    
    }
}
