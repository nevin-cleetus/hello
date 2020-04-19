pipeline {
    
    agent any
	
    
    environment
    {        
	registry = "nevincleetus/helloworld-repo"
        registryCredential = 'dockerhub'
        dockerImage = ''  
    }	
	
    tools { 
        maven 'M2_HOME'         
    }	
    stages {		
	stage ('Initialize') {
            steps {
                sh '''
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}"
                '''
            }
        }	
	    
    	/*stage ('Artifactory configuration') {
            steps {
                rtServer (
                    id: "ARTIFACTORY_ID",
                    url: SERVER_URL,
                    credentialsId: "ARTIFACTORY_CREDENTIALS"
                )

                rtMavenDeployer (
                    id: "MAVEN_DEPLOYER",
                    serverId: "ARTIFACTORY_ID",
                    releaseRepo: "hello-maven-release-local",
                    snapshotRepo: "hello-maven-snapshot-local"
                )

                rtMavenResolver (
                    id: "MAVEN_RESOLVER",
                    serverId: "ARTIFACTORY_ID",
                    releaseRepo: "hello-maven-release",
                    snapshotRepo: "hello-maven-snapshot"
                )
            }
        }*/

        stage ('Exec Maven') {
            steps {
                rtMavenRun (
                    tool: "M2_HOME", // Tool name from Jenkins configuration
                    pom: 'pom.xml',
                    goals: 'clean install',
                    deployerId: "MAVEN_DEPLOYER",
                    resolverId: "MAVEN_RESOLVER"
                )
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
                sh 'mvn clean test'
            }      
             
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
       }
	    
	    
       /*stage('SonarQube Analysis') {
	    /*agent {
               docker {
                   image 'maven:3-alpine'
                   args '-v /root/.m2:/root/.m2'
               }
            }*/
            /*steps {
              withSonarQubeEnv('SONAR_SERVER') {
                sh "mvn sonar:sonar "
	      }
           }
      } */
 
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
