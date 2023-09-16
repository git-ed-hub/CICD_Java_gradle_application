pipeline{
    agent { label 'Jenkins-Agent' } 
    tools {
        jdk 'Java11'
        gradle 'gradle'
    }
    environment{
        VERSION = "${env.BUILD_ID}"
        DOCKER_USER = "testsysadmin8"
        DOCKER_PASS = 'dockerhub'
    }
    stages{
        stage("sonar quality check"){
            
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') {
                            sh 'chmod +x gradlew'
                            sh './gradlew --warning-mode none sonarqube'
                    }

                }  
            }
        }
     stage("docker build & docker push"){
         steps{
             script{
                 withCredentials([gitUsernamePassword(credentialsId: 'dockerhub', gitToolName: 'Default')]) {
                          sh '''
                             docker build -t testsysadmin8:8083/springapp:${VERSION} .
                             docker login -u ${DOCKER_USER} -p ${DOCKER_PASS} testsysadmin8:8083 
                             docker push  testsysadmin8:8083/springapp:${VERSION}
                             docker rmi testsysadmin8:8083/springapp:${VERSION}
                            '''
                    }
                }
            }
        }    
    
    }
}
