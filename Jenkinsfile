def buildNumber = Jenkins.instance.getItem('cicd-jenkinsbeans-stage').lastSuccessfulBuild.number

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
        ARTIFACT_NAME = "vprofile-v${buildNumber}.war"
        AWS_S3_BUCKET = 'beanstalkproject'
        AWS_EB_APP_NAME = 'cicd-beanstalk'
        AWS_EB_ENVIRONMENT = 'Cicdbeanstalk-env-1'
        AWS_EB_APP_VERSION = "${buildNumber}"
    }
	
    stages{        
        stage("Deploy to Beanstalk-Staging Env") {
            steps {

                withAWS(credentials: 'awsbeans', region: 'us-east-1') {
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
