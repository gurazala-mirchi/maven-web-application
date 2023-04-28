node {
    echo "job name is: ${env.JOB_NAME}"
    echo "node name is: ${env.NODE_NAME}"
    echo "build number is: ${env.BUILD_NUMBER}"
    def mavenHome = tool name: "maven 3.9.1"
    properties([buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '5', daysToKeepStr: '', numToKeepStr: '5')), [$class: 'JobLocalConfiguration', changeReasonComment: ''], pipelineTriggers([cron(''), pollSCM('* * * * *')])])
    try{
        sendSlackNotifications('STARTED')
    stage('CheckOutCode'){
    git branch: 'development', changelog: false, credentialsId: '3b40bb9e-66b5-4fe0-843c-d52e3f399ede', poll: false, url: 'https://github.com/gurazala-mirchi/maven-web-application.git'
    }
    stage('Build'){
        sh "${mavenHome}/bin/mvn clean package"
    }
    stage('CodeQuality'){
        sh "${mavenHome}/bin/mvn sonar:sonar"
    }
    stage('Artifactory'){
        sh "${mavenHome}/bin/mvn deploy"
    }
    stage('DeploytoConatainer'){
        sshagent(['032c67db-e421-4222-991f-9eb8d79adfe8']) {
            sh "scp -o StrictHostKeyChecking=no target/maven-web-application.war ec2-user@10.42.2.30:/opt/apache-tomcat-9.0.73/webapps/"
        }
    }
    }catch (e) {
         currentBuild.result = 'FAILURE'
        throw e
    } 
    finally {
        sendSlackNotifications(currentBuild.result)
    }
}//node closing
def sendSlackNotifications(String buildStatus = 'STARTED') {
    
    buildStatus = buildStatus ?: 'SUCCESS'

    def color

    if (buildStatus == 'STARTED') {
        color = '#D4DADF'
    } else if (buildStatus == 'SUCCESS') {
        color = '#BDFFC3'
    } else if (buildStatus == 'UNSTABLE') {
        color = '#FFFE89'
    } else {
        color = '#FF9FA1'
    }

    def msg = "${buildStatus}: `${env.JOB_NAME}` #${env.BUILD_NUMBER}:\n${env.BUILD_URL}"

    slackSend(color: color, message: msg, channel: wallmart)
}
