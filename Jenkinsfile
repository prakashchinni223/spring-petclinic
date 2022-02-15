pipeline {
    agent any

    tools {
        // Install the Maven version configured as "M3" and add it to the path.
        maven "maven3"
    }
     environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "192.168.31.117:8081"
        NEXUS_REPOSITORY = "spring-petclinic"
        NEXUS_CREDENTIAL_ID = "Nexus_credentails"
    }
    stages {
	    stage('clone') {
		   steps {
		   git credentialsId: 'prakashchinni223', url: 'https://github.com/prakashchinni223/spring-petclinic.git'
		   }
		}
        stage('Build') {
            steps {
                
                sh "mvn clean package"

            }
          post {
     
                success {
                    archiveArtifacts 'target/*.war'
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }
        
        stage("publish to nexus") {
            steps {
                script {
                    pom = readMavenPom file: "pom.xml";
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    artifactPath = filesByGlob[0].path;
                    artifactExists = fileExists artifactPath;
                    if(artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}";
                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            //version: '${BUILD_NUMBER}',
                            version: pom.version,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: pom.packaging],
                                // Lets upload the pom.xml file for additional information for Transitive dependencies
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: "pom.xml",
                                type: "pom"]
                            ]
                        );
                    } else {
                        error "*** File: ${artifactPath}, could not be found";
                    }
                }
            
            }
            
        }
        
     stage("Deploy the war file in Tomcat")
        {
        steps
            {
                script{
                    
            withCredentials([usernameColonPassword(credentialsId: 'Nexus_credentails', variable: 'Nexus_cred')]) {
    sh "curl -v -u $Nexus_cred $NEXUS_PROTOCOL://$NEXUS_URL/repository/$NEXUS_REPOSITORY/org/springframework/samples/$pom.artifactId/$pom.version/$pom.artifactId-$pom.version.$pom.packaging -o result.war"
}
            withCredentials([usernameColonPassword(credentialsId: 'tomcat_credentails', variable: 'tomcat_cred')])
                       {
             sh 'curl -v -u $tomcat_cred -T "result.war" "http://192.168.31.119:8080/manager/text/deploy?path=/petclinic&update=true"' 
                        }
                }
            }
        }    
        
    }
}
