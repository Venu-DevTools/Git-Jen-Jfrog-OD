pipeline {
    agent any
 
    environment { 
        PROJECT_NAME = "allen"                         // Project name
        REPO_NAME = "odtest"                           // Repo name
        MAVEN_HOME = tool 'Maven'                      // Maven tool name in Jenkins
        ARTIFACT_NAME = "${PROJECT_NAME}_${BUILD_NUMBER}.jar"
        JFROG_REPO = "odtesting"                       // Actual JFrog repo name
        JFROG_URL = "http://130.131.164.192:8082" // JFrog Artifactory URL
        JFROG_USERNAME = "admin"               // JFrog username
        JFROG_PASSWORD = "Admin@1234"               // JFrog password
    }
 
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'Githubidvenu',
                    url: 'https://github.com/Venu-DevTools/Git-Jen-Jfrog-OD.git'
            }
        }
 
        stage('Build with Maven') {
            steps {
                sh "${MAVEN_HOME}/bin/mvn clean install -Djar.finalName=${PROJECT_NAME}_${BUILD_NUMBER}"
            }
        }
 
        stage('Archive JAR') {
            steps {
                archiveArtifacts artifacts: "target/${ARTIFACT_NAME}", fingerprint: true
            }
        }
 
        stage('Deploy to JFrog Artifactory') {
            steps {
                script {
                    // Initialize the server with the JFrog URL
                    def server = Artifactory.server(http://130.131.164.192:8082)  // Use the JFrog URL
                    def buildInfo = Artifactory.newBuildInfo()
 
                    // Get clean branch name (e.g., 'refs/heads/main' → 'main')
                    def branch = env.GIT_BRANCH?.replaceFirst(/^refs\/heads\//, '') ?: 'unknown'
 
                    // Print the upload path for debugging
                    echo "Uploading to: nets/${JFROG_REPO}/${branch}/${BUILD_NUMBER}/${ARTIFACT_NAME}"
 
                    def uploadSpec = """{
                        "files": [
                            {
                                "pattern": "target/${ARTIFACT_NAME}",
                                "target": "${JFROG_REPO}/nets/${REPO_NAME}/${branch}/${BUILD_NUMBER}/"
                            }
                        ]
                    }"""
 
                    // Use manual credentials for authentication
                    def username = $JFROG_USERNAME
                    def password = $JFROG_PASSWORD
 
                    // Set the server with credentials
                    server.setCredentials(username, password)
 
                    server.upload spec: uploadSpec, buildInfo: buildInfo
                    server.publishBuildInfo buildInfo
                }
            }
        }
    }
}
