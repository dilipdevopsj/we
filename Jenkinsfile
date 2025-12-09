node {

    def maven = "/var/lib/jenkins/tools/hudson.tasks.Maven_MavenInstallation/njob/bin/mvn"

    try {

        stage('Build Info') {
            echo "Git Branch Name: ${env.BRANCH_NAME}"
            echo "Build Number: ${env.BUILD_NUMBER}"
            notifyBuild('STARTED')
        }

        stage('Git Checkout') {
            git branch: 'dev', url: 'https://github.com/dilipdevopsj/we.git'
        }

        stage('Compile') {
            sh "${maven} compile"
        }

        stage('Build') {
            sh "${maven} clean package"
        }

        stage('SonarQube Report') {
            sh "${maven} sonar:sonar"
        }

        stage('Deploy Into Nexus') {
            sh "${maven} clean deploy"
        }

        stage('Deploy to Tomcat') {
            sh """
                curl -u sai:dd \
                --upload-file target/maven-web-application.war \
                "http://13.232.85.230:8080/manager/text/deploy?path=/maven-web-application&update=true"
            """
        }

    } catch (e) {
        currentBuild.result = "FAILED"
        throw e

    } finally {
        notifyBuild(currentBuild.result)
    }

}

def notifyBuild(String buildStatus = 'STARTED') {

  buildStatus = buildStatus ?: 'SUCCESS'

  def colorCode = '#FF0000'
  if (buildStatus == 'STARTED')   colorCode = '#FFFF00'
  if (buildStatus == 'SUCCESS')   colorCode = '#00FF00'

  def summary = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})"

  slackSend(color: colorCode, message: summary, channel: '#jo')
}
