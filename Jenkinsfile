pipeline {
    
    environment { 
        registry = "attaulbari/springclinic"
        registryCredential = 'dockerhub' 
        dockerImage = ''
        }
    
    agent any

    tools {
        maven "3.8.1" // You need to add a maven with name "3.6.0" in the Global Tools Configuration page
    }

    stages {
        stage("Build") {
            steps {
            sh 'cp settings.xml /root/.m2/'
            sh "mvn clean install -Dcheckstyle.skip"
            }
        }
        
        
        stage("upload to artifactory") {
            steps {
                rtUpload (
                    buildName: JOB_NAME,
                    buildNumber: BUILD_NUMBER,
                    serverId: SERVER_ID,
                     spec: '''{
                               "files": [
                                  {
                                    "pattern": "**/target/*.jar",
                                    "target": "result/",
                                    "recursive": "false"
                                    }
                                  ]
                              }'''  
                            )
                          }
                        }
                                  
        
    
        stage("Dcoker build") {
            steps {
                script {
                    dockerImage = docker.build registry + ":$BUILD_NUMBER" 
                    }
                }
            }
            
        stage("Deploy Image to dockerhub"){
            steps {
                script {
                    docker.withRegistry( '', registryCredential ) {
                        dockerImage.push()
                     }
                   }   
                }                 
             }
    
        stage("Image clean up") {
            steps {
                sh "docker rmi $registry:$BUILD_NUMBER" 
                   }
            }
            
    
    }    
    
    

    post {
        always {
            junit '**/target/surefire-reports/TEST-*.xml'
            cleanWs()
        }
    }
}
