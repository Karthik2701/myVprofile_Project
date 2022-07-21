pipeline {

    agent any

    parameters{
        choice(name: 'action', choices: 'create\ndestroy', description: 'Create/update or destroy the Deployment & SVC.')
        string(name: 'cluster', defaultValue : 'democluster', description: "EKS cluster name.")
        string(name: 'region', defaultValue : 'us-east-1', description: "AWS region.")
    }
    environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "172.31.31.34:8081"
        NEXUS_REPOSITORY = "vprofile-release"
	NEXUS_REPOGRP_ID    = "vpro-maven-group"
        NEXUS_CREDENTIAL_ID = "nexus_login"
        ARTVERSION = "${env.BUILD_ID}"
        ACCESS_KEY = credentials('AWS_ACCESS_KEY_ID')
        SECRET_KEY = credentials('AWS_SECRET_ACCESS_KEY')
    }

    stages{

        stage('BUILD'){
	     when {
              expression { params.action == 'create' }
          }
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }

        stage('UNIT TEST'){
	     when {
              expression { params.action == 'create' }
          }
            steps {
                sh 'mvn test'
            }
        }

        stage('INTEGRATION TEST'){
	     when {
              expression { params.action == 'create' }
           }
            steps {
                sh 'mvn verify -DskipUnitTests'
            }
        }

        stage ('CODE ANALYSIS WITH CHECKSTYLE'){
	     when {
              expression { params.action == 'create' }
            }
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }

        stage('CODE ANALYSIS with SONARQUBE') {
	    when {
              expression { params.action == 'create' }
           }

            environment {
                scannerHome = tool 'mysonarscanner4'
            }

            steps {
                withSonarQubeEnv('sonarserver') {
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile-repo \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }

                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage("Publish to Nexus Repository Manager") {
	     when {
              expression { params.action == 'create' }
           }
            steps {
                script {
                    pom = readMavenPom file: "pom.xml";
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    artifactPath = filesByGlob[0].path;
                    artifactExists = fileExists artifactPath;
                    if(artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version} ARTVERSION";
                        nexusArtifactUploader(
                                nexusVersion: NEXUS_VERSION,
                                protocol: NEXUS_PROTOCOL,
                                nexusUrl: NEXUS_URL,
                                groupId: pom.groupId,
                                version: ARTVERSION,
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
                    }
                    else {
                        error "*** File: ${artifactPath}, could not be found";
                    }
                }
            }
        }
        stage('Docker : App Image Building'){
	     when {
              expression { params.action == 'create' }
          }
            steps{
                sh """
                cd Docker-files/app/multistage/
                docker build -t vikashashoke/vprofileapp:v1.$BUILD_ID .
                docker image tag vikashashoke/vprofileapp:v1.$BUILD_ID vikashashoke/vprofileapp:latest
                """
            }
        }
//         stage('Docker : MYSQL DB Image Building'){
//             steps{
//                 sh """
//                 cd Docker-files/db/
//                 docker build -t vikashashoke/vprofiledb:v1.$BUILD_ID .
//                 docker image tag vikashashoke/vprofiledb:v1.$BUILD_ID vikashashoke/vprofiledb:latest
//                 """
//             }
//         }
//         stage('Docker : WEB(nginx) Image Building'){
//             steps{
//                 sh """
//                 cd Docker-files/web/
//                 docker build -t vikashashoke/vprofileweb:v1.$BUILD_ID .
//                 docker image tag vikashashoke/vprofileweb:v1.$BUILD_ID vikashashoke/vprofileweb:latest
//                 """
//             }
//         } 
        stage('Docker : Image push to DockerHUB '){
	     when {
              expression { params.action == 'create' }
          }
            steps{
                
                withCredentials([string(credentialsId: 'dockerHub_passwd', variable: 'docker_cred')]) {
                sh """
                
                docker login -u vikashashoke -p ${docker_cred}
                echo pushing latest app images ...
                docker image push vikashashoke/vprofileapp:v1.$BUILD_ID 
                docker image push vikashashoke/vprofileapp:latest
		
		"""
                
//                 echo pushing DB images ...
//                 docker image push vikashashoke/vprofiledb:v1.$BUILD_ID
//                 docker image push vikashashoke/vprofiledb:latest     
                
//                 echo pushing Web images ...
//                 docker image push vikashashoke/vprofileapp:v1.$BUILD_ID
//                 docker image push vikashashoke/vprofileapp:latest                
                
            }
          }
        } 
        stage('Docker : App Image Removal'){
	     when {
              expression { params.action == 'create' }
           }
            steps{
                sh """
                docker image rm vikashashoke/vprofileapp:v1.$BUILD_ID
                docker image rm vikashashoke/vprofileapp:latest
		"""
//                 docker image rm vikashashoke/vprofiledb:v1.$BUILD_ID
//                 docker image rm vikashashoke/vprofiledb:latest
//                 docker image rm vikashashoke/vprofileweb:v1.$BUILD_ID
//                 docker image rm vikashashoke/vprofileweb:latest
            }
        }
        stage('Connection to cluster'){
	    when {
              expression { params.action == 'create' }
          }
            steps{
                sh """
                aws configure set aws_access_key_id "$ACCESS_KEY"
                aws configure set aws_secret_access_key "$SECRET_KEY"
                aws configure set region "${params.region}"
                
                aws eks update-kubeconfig --name ${params.cluster} --region ${params.region}
                
                 """
            }
        } 
        stage('Deployment on Eks Cluster'){
             when {
              expression { params.action == 'create' }
          }
             steps{
                sh """
                 cd kubefiles/
                  kubectl apply -f .
                 """
                 }
        }
        stage('Delete Deployment on Eks Cluster'){
             when {
              expression { params.action == 'destroy' }
             }
             steps{
                sh """
                  cd kubefiles/
                  kubectl delete -f .
                  """
             }
        }     
    }
}





// SNAP-REPO=vprofile-snapshot
// NEXUS-USER=admin
// NEXUS-PASS=admin123
// RELEASE-REPO=vprofile-release
// CENTRAL-REPO=vpro-maven-central
// NEXUS-GRP-REPO=vpro-maven-group
// NEXUSIP=172.31.31.34
// NEXUSPORT=8081
// 62aad949f9852ec11205c13b3df2d1bd5feb56c1




