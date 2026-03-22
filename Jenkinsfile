node {

    def mavenHome = tool name: "maven-3.9.9"

    try {

        // 🔵 Build Started
        notifyBuild('STARTED')

        stage('Git Checkout') {
            notifyStage('Git Checkout')
            git branch: 'stage', url: 'https://github.com/jio-ramdevops/jio-032026.git'
        }

        stage('Maven Build') {
            notifyStage('Maven Build')
            sh "${mavenHome}/bin/mvn clean package"
        }

        stage('Sonar Scan') {
            notifyStage('Sonar Scan')
            sh "${mavenHome}/bin/mvn sonar:sonar"
        }

        stage('Nexus Deploy') {
            notifyStage('Nexus Deploy')
            sh "${mavenHome}/bin/mvn clean deploy"
        }

        stage('Deploy to Tomcat') {
            notifyStage('Tomcat Deployment')

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

        // ✅ Mark success explicitly
        currentBuild.result = 'SUCCESS'

    } catch (e) {

        // ❌ Mark failure
        currentBuild.result = 'FAILURE'
        throw e

    } finally {

        // 🔔 Always notify final status
        notifyBuild(currentBuild.result)
    }
}


// 🔔 Build Status Notification
def notifyBuild(String buildStatus = 'STARTED') {

    buildStatus = buildStatus ?: 'SUCCESS'

    def message = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})"

    def colorCode

    switch(buildStatus) {
        case 'STARTED':
            colorCode = '#FFFF00' // Yellow
            break
        case 'SUCCESS':
            colorCode = '#00FF00' // Green
            break
        case 'FAILURE':
            colorCode = '#FF0000' // Red
            break
        default:
            colorCode = '#808080' // Grey
    }

    slackSend(color: colorCode, message: message, channel: '#jio-devteam')
    slackSend(color: colorCode, message: message, channel: '#dev-target')
}


// 🔵 Stage Progress Notification
def notifyStage(String stageName) {

    def message = "IN PROGRESS: ${stageName} - Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})"

    slackSend(color: '#0000FF', message: message, channel: '#jio-devteam')
    slackSend(color: '#0000FF', message: message, channel: '#dev-target')
}
