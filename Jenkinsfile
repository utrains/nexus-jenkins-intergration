pipeline {
    triggers {
  pollSCM('* * * * *')
    }
   agent any
    tools {
  maven 'M2_HOME'
  }

   environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "198.74.52.93:8081"
        NEXUS_REPOSITORY = "utrains-nexus-pipeline"
        NEXUS_CREDENTIAL_ID = "Nexus"

        imageName = "geolocation"
        registryCredentials = "Nexus"
        registry = "198.74.52.93:8085/repository/docker-nexus-repo/"
        dockerImage = ''
    }

    stages {


         stage("Maven Build goelocation") {
            steps {
                echo 'Goelocation app...'
                dir('./'){
                    script {
                    sh "mvn package -DskipTests=true"
                    }
                }
            }
        }

         stage("Publish to Nexus Repository Manager") {
            steps {
                echo 'Publish to Nexus Repository Manager...'
                dir('./'){
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
                            version: pom.version,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: pom.packaging],
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
        }
        
        // Building Docker images
        stage("Build Docker Image"){
            steps{
                echo 'Build Docker Image'
                dir('./'){
                    script{
                        dockerImage = docker.build imageName
                    }
                }
            }
        }

        // Push Docker images to Nexus Registry
        stage("Uploading to Nexus Registry"){
            steps{
                echo 'Uploading Docker image to Nexus ...'
                dir('./'){
                    script{
                        docker.withRegistry( 'http://'+registry, registryCredentials ) {
                        dockerImage.push('latest')
                        }
                    }
                }
            }
        }   
    }
}