node {
 def mavenHome=tool name:'maven-3.9.9'
stage('git checkout')
{
git branch: 'development', url: 'https://github.com/jio-ramdevops/jio-032026.git'
}
stage('maven build')
{
 sh  "${mavenHome}/bin/mvn clean package"
}
stage ('sonar stage build')
{
 sh  "${mavenHome}/bin/mvn sonar:sonar"

}
stage ('nexus stage build')
{
 sh  "${mavenHome}/bin/mvn clean deploy"

}
stage('Deploy to Tomcat') {
    echo "Deploying WAR file using curl..."

    sh """
        curl -u sai:password \
        --upload-file /var/lib/jenkins/workspace/jio-dev-scriptedpl/target/maven-web-application.war \
        "http://13.234.122.192:8080/manager/text/deploy?path=/maven-web-application&update=true"
    """
}


}
