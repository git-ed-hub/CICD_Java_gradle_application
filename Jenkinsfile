pipeline{
    agent { label 'Jenkins-Agent' } 
    tools {
        jdk 'Java11'
        gradle 'gradle'
    }
    environment{
        VERSION = "${env.BUILD_ID}"
       
        
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
                 withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'pass', usernameVariable: 'user')]) {
                         sh '''
                             docker build -t testsysadmin8:8083/springapp:${VERSION} .
                             docker login -u testsysadmin8 -p $pass testsysadmin8:8083 
                             docker push  testsysadmin8:8083/springapp:${VERSION}
                             docker rmi testsysadmin8:8083/springapp:${VERSION}
                            '''
                    }
                 
                }
            }
        }    
    
    }
}
