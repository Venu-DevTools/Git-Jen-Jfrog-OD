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

        // Octopus deployment environment
        OCTOPUS_API_KEY = credentials('octopus-api-key')
        OCTOPUS_PROJECT = 'helloworld'
        OCTOPUS_SPACE = 'firefist'
        OCTOPUS_ENVIRONMENT = 'development'
        OCTOPUS_SERVER = 'https://devtools.octopus.app/'
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

        stage('Archive JAR') {
            steps {
                archiveArtifacts artifacts: "target/${ARTIFACT_NAME}", fingerprint: true
            }
        }

        stage('Create Release in Octopus') {
            steps {
                withEnv(["OCTO_API_KEY=${OCTOPUS_API_KEY}"]) {
                    sh '''
                        octo create-release \
                        --project "${OCTOPUS_PROJECT}" \
                        --version ${PACKAGE_VERSION} \
                        --server ${OCTOPUS_SERVER} \
                        --apiKey ${OCTO_API_KEY} \
                        --space "${OCTOPUS_SPACE}" \
                        --packageVersion ${PACKAGE_VERSION}
                    '''
                }
            }
        }

        stage('Deploy Release to Environment') {
            steps {
                withEnv(["OCTO_API_KEY=${OCTOPUS_API_KEY}"]) {
                    sh '''
                        octo deploy-release \
                        --project "${OCTOPUS_PROJECT}" \
                        --version ${PACKAGE_VERSION} \
                        --server ${OCTOPUS_SERVER} \
                        --apiKey ${OCTO_API_KEY} \
                        --space "${OCTOPUS_SPACE}" \
                        --deployTo "${OCTOPUS_ENVIRONMENT}" \
                        --progress
                    '''
                }
            }
        }
    }
}
