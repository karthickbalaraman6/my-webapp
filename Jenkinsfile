pipeline {
agent any


environment {
// Credentials IDs stored in Jenkins
GIT_CRED = 'git-cred-id'
NEXUS_CRED = 'nexus-cred-id' // username/password
TOMCAT_CRED = 'tomcat-cred-id' // username/password for Tomcat manager
TOMCAT_URL = 'http://tomcat-server:8080'
SONAR_SERVER = 'SonarQube' // used if SonarQube plugin is configured
MVN_HOME = tool 'Maven 3' // name of Maven installer in Jenkins tools config
}


stages {
stage('Checkout') {
steps {
checkout scm
}
}


stage('Build') {
steps {
withEnv(["PATH+MAVEN=${env.MVN_HOME}/bin"]) {
sh 'mvn -B clean package -DskipTests'
}
}
}


stage('SonarQube Analysis') {
steps {
// If using SonarQube plugin in Jenkins, use 'withSonarQubeEnv'
withSonarQubeEnv('SonarQube') {
sh '${MVN_HOME}/bin/mvn -B sonar:sonar'
}
}
}


stage('Publish to Nexus') {
steps {
withCredentials([usernamePassword(credentialsId: NEXUS_CRED, usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASSWORD')]) {
// write settings.xml dynamically to use credentials
writeFile file: 'settings.xml', text: """
<settings>
<servers>
<server>
<id>nexus-releases</id>
<username>${NEXUS_USER}</username>
<password>${NEXUS_PASSWORD}</password>
</server>
<server>
<id>nexus-snapshots</id>
<username>${NEXUS_USER}</username>
<password>${NEXUS_PASSWORD}</password>
</server>
</servers>
</settings>
"""
sh "${MVN_HOME}/bin/mvn -B deploy -s settings.xml -DskipTests"
}
}
}


stage('Deploy to Tomcat') {
steps {
withCredentials([usernamePassword(credentialsId: TOMCAT_CRED, usernameVariable: 'TOMCAT_USER', passwordVariable: 'TOMCAT_PASS')]) {
script {
def war = "target/${env.BUILD_ID == null ? 'my-webapp-1.0.0-SNAPSHOT.war' : 'my-webapp-' + env.BUILD_ID + '.war'}"
// Better approach: reference artifact by artifactId-version.war
sh "curl -u ${TOMCAT_USER}:${TOMCAT_PASS} --upload-file target/*.war \"${TOMCAT_URL}/manager/text/deploy?path=/mywebapp&update=true\""
}
}
}
}