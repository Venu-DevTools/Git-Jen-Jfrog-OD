pipeline {
    agent any

    environment {
        // Common build environment
        PACKAGE_VERSION = "1.0.${BUILD_NUMBER}"  
        MAVEN_HOME = tool 'Maven3'
        JFROG_REPO = 'simple-local'
        ORG_PATH = 'MyProject'
        MODULE = 'hello-world' 
        ARTIFACT_NAME = "${MODULE}-${PACKAGE_VERSION}.jar"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', credentialsId: 'github-creds', url: 'https://github.com/Venu-DevTools/Git-Jen-Jfrog-OD.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh '${MAVEN_HOME}/bin/mvn clean install -Djar.finalName=${MODULE}-${PACKAGE_VERSION}'
            }
        }

        stage('Deploy to Artifactory') {
            steps {
                script {
                    def server = Artifactory.server 'Jfserver'
                    def buildInfo = Artifactory.newBuildInfo()

                    def uploadSpec = """{
                        "files": [
                            {
                                "pattern": "target/${ARTIFACT_NAME}",
                                "target": "${JFROG_REPO}/${ORG_PATH}/"
                            }
                        ]
                    }"""

                    server.upload spec: uploadSpec, buildInfo: buildInfo
                    server.publishBuildInfo buildInfo
                }
            }
        }

        stage('Push Build Info to Octopus') {
            steps {
                octopusPushBuildInformation(
                    serverId: 'octopus-server',       // Your Octopus Deploy Jenkins config ID
                    spaceId: 'firefist',              // Your Octopus space
                    toolId: 'OctoCLI',                // Your Octopus CLI tool ID
                    packageId: 'hello-world',         // Your package ID (must match the artifact ID)
                    packageVersion: "${PACKAGE_VERSION}"
                )
            }
        }

        stage('Archive JAR') {
            steps {
                archiveArtifacts artifacts: "target/${ARTIFACT_NAME}", fingerprint: true
            }
        }
    }
}
