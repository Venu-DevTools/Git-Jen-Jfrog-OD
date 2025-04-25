pipeline {
    agent any

    environment {
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

        stage('Generate Build Info JSON') {
            steps {
                script {
                    // Generate build info JSON dynamically including Git commit hash
                    writeFile file: 'build-info.json', text: """{
                        "Version": "${PACKAGE_VERSION}",
                        "PackageId": "MyProject/hello-world",
                        "BuildEnvironment": "Jenkins",
                        "BuildNumber": "${env.BUILD_NUMBER}",
                        "Branch": "${env.GIT_BRANCH ? env.GIT_BRANCH : 'main'}",
                        "VcsType": "Git",
                        "VcsRoot": "https://github.com/Venu-DevTools/Git-Jen-Jfrog-OD.git",
                        "VcsCommitNumber": "${env.GIT_COMMIT}",
                        "ProjectID": "zeta",
                        "System": "Lota",
                        "Service": "Kappa"
                    }"""
                }
            }
        }

        stage('Push Build Info to Octopus') {
            steps {
                script {
                    // Now push the generated JSON file to Octopus
                    octopusPushBuildInformation(
                        serverId: 'octopus-server',  // Your Octopus server ID
                        spaceId: 'firefist',         // Your Octopus space ID
                        toolId: 'OctoCLI',           // Octopus CLI tool ID
                        packageId: 'MyProject/hello-world',
                        packageVersion: "${PACKAGE_VERSION}",
                        commentParser: '',
                        overwriteMode: 'OverwriteExisting',
                        additionalArgs: '--file="build-info.json"'
                    )
                }
            }
        }

        stage('Archive JAR') {
            steps {
                archiveArtifacts artifacts: "target/${ARTIFACT_NAME}", fingerprint: true
            }
        }
    }
}

