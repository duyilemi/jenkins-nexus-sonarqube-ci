pipeline {
    
	agent any
    tools {
        maven "MAVEN3"
        jdk "OracleJDK8"
    }
    
    environment {
        SNAP_REPO = 'vprofile-snapshot'
        NEXUS_USER = 'admin'
        NEXUS_PASS = 'nexus'
        RELEASE_REPO = 'vprofile-release'
        CENTRAL_REPO = 'vpro-maven-central'
        NEXUS_IP = '172.31.29.250'
        NEXUS_PORT = '8081'
        NEXUS_GRP_REPO = 'vpro-maven-group'
        NEXUS_LOGIN = 'nexus'
        SONARSCANNER = 'sonarscanner'
        SONARSERVER = 'sonarserver'
    }
	
    stages{
        
        stage('Build'){
            steps {
                sh 'mvn -s settings.xml -DskipTests install'
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }

	    stage('Unit Test'){
            steps {
                sh 'mvn -s settings.xml test'
            }
        }

	// stage('INTEGRATION TEST'){
    //         steps {
    //             sh 'mvn verify -DskipUnitTests'
    //         }
    //     }
		
        stage ('Code Analysis With Checkstyle'){
            steps {
                sh 'mvn -s settings.xml checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }

        stage('Code Analysis With SonarQube') {
          
		  environment {
             scannerHome = tool "${SONARSCANNER}"
          }

          steps {
            withSonarQubeEnv("${SONARSERVER}") {
               sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile-repo \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
            }


          }
        }

        stage('Sonarqube Quality Gates') {
            steps {
            timeout(time: 10, unit: 'MINUTES') {
               waitForQualityGate abortPipeline: true
            }
            }
        }

        stage('Publish Artifacts to Nexus Repository Manager') {
            steps {
                 nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: "${NEXUS_IP}:${NEXUS_PORT}",
                    groupId: 'QA',
                    version: "${env.BUILD_ID}:${BUILD_TIMESTAMP}",
                    repository: "${RELEASE_REPO}",
                    credentialsId: "${NEXUS_LOGIN}",
             artifacts: [
                [artifactId: vproapp,
             classifier: '',
             file: 'target/vprofile-v2.war',
             type: 'war']
        ]
     )
            }
        }

    //     stage("Publish to Nexus Repository Manager") {
    //         steps {
    //             script {
    //                 pom = readMavenPom file: "pom.xml";
    //                 filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
    //                 echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
    //                 artifactPath = filesByGlob[0].path;
    //                 artifactExists = fileExists artifactPath;
    //                 if(artifactExists) {
    //                     echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version} ARTVERSION";
    //                     nexusArtifactUploader(
    //                         nexusVersion: NEXUS_VERSION,
    //                         protocol: NEXUS_PROTOCOL,
    //                         nexusUrl: NEXUS_URL,
    //                         groupId: NEXUS_REPOGRP_ID,
    //                         version: ARTVERSION,
    //                         repository: NEXUS_REPOSITORY,
    //                         credentialsId: NEXUS_CREDENTIAL_ID,
    //                         artifacts: [
    //                             [artifactId: pom.artifactId,
    //                             classifier: '',
    //                             file: artifactPath,
    //                             type: pom.packaging],
    //                             [artifactId: pom.artifactId,
    //                             classifier: '',
    //                             file: "pom.xml",
    //                             type: "pom"]
    //                         ]
    //                     );
    //                 } 
	// 	    else {
    //                     error "*** File: ${artifactPath}, could not be found";
    //                 }
    //             }
    //         }
    //     }


    }


}
