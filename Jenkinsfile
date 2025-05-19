pipeline {
    agent any

    environment { 
        PROJECT_NAME = "allen"                         // Project name
        REPO_NAME="odtest"                               //Repo name
        MAVEN_HOME = tool 'Maven'                      // Maven tool name in Jenkins
        ARTIFACT_NAME = "${PROJECT_NAME}_${BUILD_NUMBER}.jar"
        JFROG_REPO = "odtesting"                       // Actual JFrog repo name
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
                    def server = Artifactory.server('JFrog')  // JFrog instance ID
                    def buildInfo = Artifactory.newBuildInfo()

                    // Get clean branch name (e.g., 'refs/heads/main' → 'main')
                    def branch = env.GIT_BRANCH?.replaceFirst(/^refs\/heads\//, '') ?: 'unknown'

                    // Print the upload path for debugging
                    echo "Uploading to: nets/${JFROG_REPO}/${branch}/${BUILD_NUMBER}/${ARTIFACT_NAME}"

                    def uploadSpec = """{
                        "files": [
                            {
                                "pattern": "target/${ARTIFACT_NAME}",
                                "target": "${JFROG_REPO}/nets/${REPO_NAME}/${branch}/${BUILD_NUMBER}/${ARTIFACT_NAME}"
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
