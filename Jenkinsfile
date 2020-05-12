//START-OF-SCRIPT
node {
    def server = Artifactory.server 'artifactory'
    def SONARQUBE_HOSTNAME = 'sonarqube'
    def SPLUNK_HOSTNAME='splunk'
    def DOCKER_HOME = tool name: 'docker-latest'
    def GRADLE_HOME = tool name: 'gradle-4.10.2', type: 'hudson.plugins.gradle.GradleInstallation'
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
    
    stage('Build') {
        sh "${GRADLE_HOME}/bin/gradle build"
        sh "ls -la build/libs/*.war"
        sh "echo ====================="
        sh "cp build/libs/*.war docker/webapp.war"
        sh "pwd"
        sh "ls -la"
        sh "ls -la ./docker"
    }
    
    stage ('Docker Build') {
        docker.withTool('docker-latest') {
            sh "printenv"
            sh "pwd"
            sh "ls -la"
            def image1 = docker.build("${DOCKERHUB_REPO}:${BUILD_NUMBER}", "./docker")
            def image2 = docker.build("${DOCKERHUB_REPO}:latest", "./docker")
        }
    }
    
    stage('sonar-scanner') {
      def sonarqubeScannerHome = tool name: 'sonar', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
      withCredentials([string(credentialsId: 'sonar', variable: 'sonarLogin')]) {
        sh "${sonarqubeScannerHome}/bin/sonar-scanner -e -Dsonar.host.url=http://${SONARQUBE_HOSTNAME}:9000 -Dsonar.login=${sonarLogin} -Dsonar.projectName=WebApp -Dsonar.projectVersion=${env.BUILD_NUMBER} -Dsonar.projectKey=GS -Dsonar.sources=src/main/ -Dsonar.tests=src/test/ -Dsonar.java.binaries=build/**/* -Dsonar.language=java"
      }
    }
}
//END-OF-SCRIPT
