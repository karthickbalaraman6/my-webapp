pipeline {
    agent any

    environment {
        GIT_CRED     = 'git-cred-id'
        NEXUS_CRED   = 'nexus-cred-id'
        TOMCAT_CRED  = 'tomcat-cred-id'
        TOMCAT_URL   = 'http://tomcat-server:8080'
        SONAR_SERVER = 'SonarQube'
        MVN_HOME     = tool 'Maven 3'
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
                withSonarQubeEnv('SonarQube') {
                    sh "${MVN_HOME}/bin/mvn -B sonar:sonar"
                }
            }
        }

        stage('Publish to Nexus') {
            steps {
                withCredentials([usernamePassword(credentialsId: NEXUS_CRED, usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASSWORD')]) {

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

                    sh """
                        curl -u ${TOMCAT_USER}:${TOMCAT_PASS} --upload-file target/*.war \
                        "${15.206.168.45:8083}/manager/text/deploy?path=/mywebapp&update=true"
                    """
                }
            }
        }
    }

    post {
        success {
            echo "BUILD SUCCESS"
        }
        failure {
            echo "BUILD FAILED"
        }
    }
}