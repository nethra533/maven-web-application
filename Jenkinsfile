node
{
def mavenHome = tool name: "maven3.8.5"
properties([buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '5', daysToKeepStr: '', numToKeepStr: '5')), [$class: 'JobLocalConfiguration', changeReasonComment: ''], pipelineTriggers([pollSCM('* * * * *')])])

echo "The Job name is: ${env.JOB_NAME}"
echo "The node name is: ${env.NODE_NAME}"
echo "The Workspace is: ${env.WORKSPACE}"
echo "The node label is: ${env.NODE_LABELS}"
echo "The build number is: ${env.BUILD_NUMBER}"

try{
notifyBuild("STARTED")  
        
stage('CheckoutCode'){
git branch: 'development', credentialsId: 'b6527bb1-1c6b-4e35-be6e-ac36de1fcd4d', url: 'https://github.com/nethra533/maven-web-application.git'
}
stage('Build'){
sh "${mavenHome}/bin/mvn clean package"
}
stage('ExecuteSonarQubeReport'){
sh "${mavenHome}/bin/mvn sonar:sonar"    
}
stage('UploadArtifactsIntoNexus'){
sh "${mavenHome}/bin/mvn deploy"
}   
stage('DeployAppIntoTomcatServer'){
sshagent(['2bcbc7a5-054b-47f5-bb80-190571314377']) {
sh "scp -o StrictHostKeyChecking=no target/maven-web-application.war ec2-user@65.0.31.133:/opt/apache-tomcat-9.0.65/webapps/"
}
}  
}   
catch(e){
currentBuild.result = "FAILED"
throw e
}
finally{ 
notifyBuild(currentBuild.result)
}   

}  //node closing
    
def notifyBuild(String buildStatus = 'STARTED') {
  // build status of null means successful
  buildStatus =  buildStatus ?: 'SUCCESSFUL'

  // Default values
  def colorName = 'RED'
  def colorCode = '#FF0000'
  def subject = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
  def summary = "${subject} (${env.BUILD_URL})"

  // Override default values based on build status
  if (buildStatus == 'STARTED') {
    color = 'ORANGE'
    colorCode = '#FFA500'
  } else if (buildStatus == 'SUCCESSFUL') {
    color = 'GREEN'
    colorCode = '#00FF00'
  } else {
    color = 'RED'
    colorCode = '#FF0000'
  }

  // Send notifications
  slackSend (color: colorCode, message: summary)
}
