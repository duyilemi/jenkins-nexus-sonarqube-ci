def COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger'
]

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
        ARTIFACT_NAME = "vprofile-v${BUILD_ID}.war"
        AWS_S3_BUCKET = 'beanstalkproject'
        AWS_EB_APP_NAME = 'cicd-beanstalk'
        AWS_EB_ENVIRONMENT = 'Cicdbeanstalk-env'
        AWS_EB_APP_VERSION = "${BUILD_ID}"
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
                [artifactId: 'vproapp',
             classifier: '',
             file: 'target/vprofile-v2.war',
             type: 'war']])
            }
        }

        stage("Deploy to Beanstalk-Staging Env") {
            steps {

                withAWS(credentials: 'awsbeans', region: 'us-east-1') {
                    sh 'aws s3 cp ./target/vprofile-v2.war s3://$AWS_S3_BUCKET/$ARTIFACT_NAME'
                    sh 'aws elasticbeanstalk create-application-version --application-name $AWS_EB_APP_NAME --version-label $AWS_EB_APP_VERSION --source-bundle S3Bucket=$AWS_S3_BUCKET, S3Key=$ARTIFACT_NAME'
                    sh 'aws elasticbeanstalk update-environment --application-name $AWS_EB_APP_NAME --environment-name $AWS_EB_ENVIRONMENT --version-label $AWS_EB_APP_VERSION'
                }
            }
        }

    }

    
    // post {
    //     always {
    //         echo 'Slack Notifications.'
    //         slackSend channel: '#jenkinsslack',
    //             color: COLOR_MAP[currentBuild.currenrResult],
    //             message: "*${currentBuild.currenrResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n get more info at:${env.BUILD_URL}"
    //     }
    // }

}
