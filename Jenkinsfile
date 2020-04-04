pipeline {
    
    agent any
	
    tools {
       mvn_version = 'M2_HOME'    
    }	

    environment
    {        
	registry = "nevincleetus/java-web-app-cicd"
        registryCredential = 'dockerhub'
        dockerImage = ''  
    }	
   
	
    stages {
    	stage('Build') {
	    steps {
               sh 'mvn -B -DskipTests clean package'
            }    
            
       }
       stage('Test') {
            
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
       }
       stage('Building image') {
          steps{
              script {
                  dockerImage = docker.build registry + ":$BUILD_NUMBER"
              }
          }
      }	 
	    
      stage('Deploy Image') {
         steps{
            script {
                docker.withRegistry( '', registryCredential ) {
                dockerImage.push()
            }
          }
        }
     }
     stage('Remove Unused docker image') {
        steps{
          sh "docker rmi $registry:$BUILD_NUMBER"
        }
      }    
    }	
}
