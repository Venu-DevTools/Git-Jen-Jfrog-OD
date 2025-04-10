pipeline {
    agent any

    environment {
        PACKAGE_VERSION = "1.0.${BUILD_NUMBER}"
        MAVEN_HOME = tool 'Maven3'
        JFROG_CLI_HOME = tool 'JFrogCLI' // Ensure 'jfrog-cli' is configured in Jenkins Global Tool Configuration
        JFROG_SERVER_ID = 'jfrog-server'  // Ensure this matches the server ID configured in JFrog CLI
        JFROG_REPO = 'simple-local'       // Target Artifactory repository
        ORG_PATH = 'MyProject'            // Organization path
        MODULE = 'hello-world'            // Module name
        ARTIFACT_NAME = "${MODULE}-${PACKAGE_VERSION}.jar"
        JFROG_CLI_BUILD_NAME = "${JOB_NAME}"
        JFROG_CLI_BUILD_NUMBER = "${BUILD_NUMBER}"  
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', credentialsId: 'github-creds', url: 'https://github.com/Venu-DevTools/Git-Jen-Jfrog-OD.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh '${MAVEN_HOME}/bin/mvn clean package -Djar.finalName=${MODULE}-${PACKAGE_VERSION}'
            }
        }

        stage('Archive JAR') {
            steps {
                archiveArtifacts artifacts: "target/${ARTIFACT_NAME}", fingerprint: true
            }
        }

        stage('Upload to JFrog Artifactory') {
            steps {
                script {
                    // Set JFrog CLI executable path
                    def jfrogCli = "${JFROG_CLI_HOME}/jf"

                    // Upload the artifact to Artifactory
                    sh '${jfrogCli} rt upload --server-id=${JFROG_SERVER_ID} target/${ARTIFACT_NAME} ${JFROG_REPO}/${ORG_PATH}/${MODULE}-${PACKAGE_VERSION}.jar'

                    // Publish build information to Artifactory
                    sh '${jfrogCli} rt build-publish --server-id=${JFROG_SERVER_ID} ${JFROG_CLI_BUILD_NAME} ${JFROG_CLI_BUILD_NUMBER}'
                }
            }
        }
    }
}
