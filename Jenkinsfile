//START-OF-SCRIPT
node {
    def server = Artifactory.server 'artifactory'
    def SONARQUBE_HOSTNAME = 'sonarqube'
    def JIRA_SITE_NAME = 'jira'
    def JIRA_PROJ_NAME = 'SKYNET'
    
    def GRADLE_HOME = tool name: 'gradle-4.10.2'
    def REPO_URL = 'https://github.com/cloudacademy/devops-webapp.git'
    def DOCKERHUB_REPO = 'cloudacademydevops/webapp'
  
    withCredentials([usernamePassword(credentialsId: 'artifactory',
                     usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
        server.username = "${USERNAME}"
        server.password = "${PASSWORD}"
    }    
    
    def rtGradle = Artifactory.newGradleBuild()
    def buildInfo

    stage ('Clone') {
        git url: 'https://github.com/FarouqMousa/devops-webapp.git'
    }

    stage ('Artifactory Configuration') {
        rtGradle.tool = "gradle-5.6.4" // Tool name from Jenkins configuration
        rtGradle.deployer repo: 'gradle-release-local', server: server
        rtGradle.resolver repo: 'jcenter', server: server
    }

    stage ('Gradle Build') {
        buildInfo = rtGradle.run rootDir: "./", buildFile: 'build.gradle', tasks: 'clean build'
    }

    stage ('Gradle Publish') {
        buildInfo = rtGradle.run rootDir: "./", buildFile: 'build.gradle', tasks: 'artifactoryPublish'
    }

    stage ('Artifactory Publish Build Info') {
        server.publishBuildInfo buildInfo
    }
    
    stage('Build with Gradle 4.10.2') {
        sh "${GRADLE_HOME}/bin/gradle build --info 2>&1 | tee gradle.build.${BUILD_NUMBER}.log"
        sh "ls -la build/libs/*.war"
    }
    
    stage('sonar-scanner') {
      def sonarqubeScannerHome = tool name: 'sonar', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
      withCredentials([string(credentialsId: 'sonar', variable: 'sonarLogin')]) {
        sh "${sonarqubeScannerHome}/bin/sonar-scanner -e -Dsonar.host.url=http://${SONARQUBE_HOSTNAME}:9000 -Dsonar.login=${sonarLogin} -Dsonar.projectName=WebApp -Dsonar.projectVersion=${env.BUILD_NUMBER} -Dsonar.projectKey=GS -Dsonar.sources=src/main/ -Dsonar.tests=src/test/ -Dsonar.java.binaries=build/**/* -Dsonar.language=java"
      }
    }
    
    stage('Raise JiraIssue') {
        def issue = [fields: [ project: [key: JIRA_PROJ_NAME],
                       summary: 'New JIRA Created from Jenkins.',
                       description: 'New JIRA Created from Jenkins.',
                       issuetype: [name: 'Task']]]
        
        def newIssue = jiraNewIssue issue: issue, site: JIRA_SITE_NAME
        
        def newIssueId = newIssue.data.key
        echo newIssueId
        
        def attachment1 = jiraUploadAttachment site: JIRA_SITE_NAME, idOrKey: newIssueId, file: "gradle.build.${BUILD_NUMBER}.log"
    }
}
//END-OF-SCRIPT
