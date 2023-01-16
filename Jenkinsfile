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

        // imageName = "fastfood"
        // registryCredentials = "Nexus"
        // registry = "139.177.192.139:8085/repository/utrains-nexus-registry/"
        // dockerImage = ''

        //Declare the variable version
        POM_VERSION = ''
        BUILD_NUM = "${env.BUILD_ID}"

    }

    stages {

      

    

         stage("Maven Build Back-End") {
            steps {
                echo 'Build Back-End Project...'
                script {
                sh "mvn package -DskipTests=true -Dspring.profiles.active=test"
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
        
        // // Building Docker images
        // stage("Build Docker Image"){
        //     steps{
        //         echo 'Build Docker Image'
        //         dir('./fastfood_backend/'){
        //             script{
        //                 dockerImage = docker.build imageName
        //             }
        //         }
        //     }
        // }

        // // Push Docker images to Nexus Registry
        // stage("Uploading to Nexus Registry"){
        //     steps{
        //         echo 'Uploading Docker image to Nexus ...'
        //         dir('./fastfood_backend/'){
        //             script{
        //                 pom = readMavenPom file: "pom.xml";
        //                 POM_VERSION = pom.version
        //                 docker.withRegistry( 'http://'+registry, registryCredentials ) {
                        
        //                 //Tag the Docker Image  with the version that is in the pom file.
        //                 // This version is often changed before a new release. 
        //                 dockerImage.push("${POM_VERSION}")
        //                 }
        //             }
        //         }
        //     }
        // }

        // //Backend Helm Chart push as tgz file
        // stage("pushing the Backend helm charts to nexus"){
        //     steps{
        //         script{
        //             withCredentials([string(credentialsId: 'nexus-pass', variable: 'docker_password')]) {
        //                   dir('fastfood_backend/') {
        //                      sh '''
        //                          helmversion=$( helm show chart helm_fastfood_back | grep version | cut -d: -f 2 | tr -d ' ')
        //                          tar -czvf  helm_fastfood_back-${helmversion}.tgz helm_fastfood_back/
        //                          curl -u jenkins-user:$docker_password http://139.177.192.139:8081/repository/fastfood-helm-rep/ --upload-file helm_fastfood_back-${helmversion}.tgz -v
        //                     '''
        //                   }
        //             }
        //         }
        //     }
        // }

        // //Frontend Helm Chart push as tgz file
        // stage("pushing the Frontend helm charts to nexus"){
        //     steps{
        //         script{
        //             withCredentials([string(credentialsId: 'nexus-pass', variable: 'docker_password')]) {
        //                   dir('fastfood_frontend/') {
        //                      sh '''
        //                          helmversion=$( helm show chart helm_fastfood_back | grep version | cut -d: -f 2 | tr -d ' ')
        //                          tar -czvf  helm_fastfood_front-${helmversion}.tgz helm_fastfood_front/
        //                          curl -u jenkins-user:$docker_password http://139.177.192.139:8081/repository/fastfood-helm-rep-front/ --upload-file helm_fastfood_front-${helmversion}.tgz -v
        //                     '''
        //                   }
        //             }
        //         }
        //     }
        // }      
    }
}