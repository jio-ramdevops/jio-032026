node{

    echo "git branch name: ${env.BRANCH_NAME}"
    echo "build number is: ${env.BUILD_NUMBER}"
    echo "node name is: ${env.NODE_NAME}"
    def mavenHome=tool name: "maven-3.9.9"
	
	try
	{
    stage('git checkout')
    {
    git branch: 'stage', url: 'https://github.com/jio-ramdevops/jio-032026.git'
    }
    stage('maven build')
    {
    sh "${mavenHome}/bin/mvn clean package"
    }
    stage('sonar report')
    {
    sh "${mavenHome}/bin/mvn sonar:sonar"
    }
    stage('Nexus report')
    {
    sh "${mavenHome}/bin/mvn clean deploy"
    }
    stage('Deploy to Tomcat') {
    echo "Deploying WAR file using curl..."
    withCredentials([usernamePassword(
        credentialsId: 'tomcat-creds',
        usernameVariable: 'USER',
        passwordVariable: 'PASS'
    )]) {

        sh """
            curl -u $USER:$PASS \
            --upload-file ${WORKSPACE}/target/maven-web-application.war \
            "http://13.201.119.213:8080/manager/text/deploy?path=/maven-web-application&update=true"
        """
    }
}
}
catch (e) {
    // If there was an exception thrown, the build failed
    currentBuild.result = "FAILED"
    throw e
  }
  finally {
    // Success or failure, always send notifications
    notifyBuild(currentBuild.result)
  }
  
}
def notifyBuild(String buildStatus='STARTED'){
buildStatus= buildStatus?: 'SUCCESS'
def subject = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
def summary = "${subject} (${env.BUILD_URL})"

// Override default values based on build status
  if (buildStatus == 'STARTED') {
    color = 'YELLOW'
    colorCode = '#FFFF00'
  } else if (buildStatus == 'SUCCESS') {
    color = 'GREEN'
    colorCode = '#00FF00'
  } else {
    color = 'RED'
    colorCode = '#FF0000'
  }

 // Send notifications
  slackSend (color: colorCode, message: summary, channel: '#jio-devteam')
  slackSend (color: colorCode, message: summary, channel: '#dev-target')
}
