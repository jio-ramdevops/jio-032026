node{
    def mavenHome=tool name: "maven-3.9.9"
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
