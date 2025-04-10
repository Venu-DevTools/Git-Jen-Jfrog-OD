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
                    def server = Artifactory.server 'Jfserver' // Server ID from Jenkins > Configure System

                    def buildInfo = Artifactory.newBuildInfo()

                    def uploadSpec = '''{
                        "files": [
                            {
                                "pattern": "target/${ARTIFACT_NAME}",
                                "target": "${JFROG_REPO}/${ORG_PATH}/"
                            }
                        ]
                    }'''

                    server.upload spec: uploadSpec, buildInfo: buildInfo
                    server.publishBuildInfo buildInfo
                }
            }
        }
    }
}
