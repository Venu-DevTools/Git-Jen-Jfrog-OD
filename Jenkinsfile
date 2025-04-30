pipeline {
    agent any

    environment { 
        PROJECT_NAME = "allen"                         // Project name (from Bitbucket)
        MAVEN_HOME = tool 'Maven'                      // Maven tool configured in Jenkins
        ARTIFACT_NAME = "${PROJECT_NAME}_${BUILD_NUMBER}.jar"  // Artifact name
        JFROG_REPO = "gumon"                           // JFrog repository name
    }

    stages {
        stage('Checkout') { 
            steps {
                git branch: 'main',
                    credentialsId: 'github-creds',         // Bitbucket credentials ID
                    url: 'https://github.com/Venu-DevTools/Git-Jen-Jfrog-OD.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh "${MAVEN_HOME}/bin/mvn clean install -Djar.finalName=${PROJECT_NAME}.${BUILD_NUMBER}"
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
                    def server = Artifactory.server('JFrog')  // Jenkins JFrog instance ID
                    def buildInfo = Artifactory.newBuildInfo()

                    def uploadSpec = """{
                        "files": [
                            {
                                "pattern": "target/${ARTIFACT_NAME}",
                                "target": "${JFROG_REPO}/"
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
