pipeline {
    agent any
 
    environment { 
        PROJECT_NAME = "allen"                            // Project name
        REPO_NAME = "odtest"                              // Folder-style repo path prefix
        MAVEN_HOME = tool 'Maven'                         // Maven tool name in Jenkins
        ARTIFACT_NAME = "${PROJECT_NAME}_${BUILD_NUMBER}.jar"
        JFROG_REPO = "odtesting"                          // Actual JFrog repo name
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
                    def server = Artifactory.server('JFrog')  // Jenkins Artifactory server ID
                    def buildInfo = Artifactory.newBuildInfo()
 
                    // Clean branch name from env.GIT_BRANCH
                    def rawBranch = env.GIT_BRANCH ?: 'origin/main'
                    def branch = rawBranch.replaceAll(/^refs\/heads\//, '').replaceAll(/^origin\//, '')
 
                    echo "Uploading to: nets/${REPO_NAME}/${branch}/${BUILD_NUMBER}/${ARTIFACT_NAME}"
 
                    def uploadSpec = """{
                        "files": [
                            {
                                "pattern": "target/${ARTIFACT_NAME}",
                                "target": "${JFROG_REPO}/nets/${REPO_NAME}/${branch}/${BUILD_NUMBER}/"
                            }
                        ]
                    }"""
 
                    server.upload spec: uploadSpec, buildInfo: buildInfo
                    server.publishBuildInfo buildInfo
                }
            }
        }
    }
}
