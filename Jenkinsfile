pipeline {
    
    agent any
	
    
    environment
    {        
	registry = "nevincleetus/helloworld-repo"
        registryCredential = 'dockerhub'
        dockerImage = ''  
    }	
    stages {
	
    	stage('Build') {
          agent {
            docker {
               image 'maven:3-alpine'
               args '-v /root/.m2:/root/.m2'
            }
          }

          steps {
                sh 'mvn clean package'
          }
       }
       stage('Test') {
            agent {
               docker {
                   image 'maven:3-alpine'
                   args '-v /root/.m2:/root/.m2'
               }
            }
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
       }
	    
	    
       stage('SonarQube Analysis') {
	    agent {
               docker {
                   image 'maven:3-alpine'
                   args '-v /root/.m2:/root/.m2'
               }
            }
            steps {
              withSonarQubeEnv('SONAR_SERVER') {
                sh "mvn sonar:sonar -Dsonar.projectKey=MyHello -Dsonar.host.url=http://13.232.223.233:9000/sonar -Dsonar.login=d4b7a84c8da51ff1a211392a5e99344a9e0384e7"      
              }
           }
      } 
 
      /*stage("Quality Gate Status Check") {
            steps {
              timeout(time: 1, unit: 'HOURS') {
                waitForQualityGate abortPipeline: true
              }
            }
      }*/	    
	    
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
