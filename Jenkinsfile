pipeline {
    agent any

    environment {
        PACKAGE_VERSION = "1.0.${BUILD_NUMBER}"
        MAVEN_HOME = tool 'Maven3'
        JFROG_CLI_PATH = '/usr/local/bin/jf' // Path to JFrog CLI
        JFROG_SERVER_ID = 'jfrog-server'     // JFrog server ID configured in Jenkins
        JFROG_REPO = 'simple-local'          // Target Artifactory repository
        ORG_PATH = 'MyProject'               // Organization path
        MODULE = 'hello-world'               // Module name
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
                sh "${MAVEN_HOME}/bin/mvn clean package -Djar.finalName=${MODULE}-${PACKAGE_VERSION}"
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
                    // Configure JFrog CLI with the server ID if not already configured
                    sh """
                        if ! ${JFROG_CLI_PATH} config show ${JFROG_SERVER_ID} > /dev/null 2>&1; then
                            ${JFROG_CLI_PATH} config add ${JFROG_SERVER_ID} --interactive=false
                        fi
                    """

                    // Upload the artifact to Artifactory using the specified layout
                    sh """
                        ${JFROG_CLI_PATH} rt upload \
                        --server-id=${JFROG_SERVER_ID} \
                        --build-name=${JOB_NAME} \
                        --build-number=${BUILD_NUMBER} \
                        "target/${ARTIFACT_NAME}" \
                        "${JFROG_REPO}/${ORG_PATH}/${MODULE}-${PACKAGE_VERSION}.jar"
                    """

                    // Publish build information to Artifactory
                    sh "${JFROG_CLI_PATH} rt build-publish ${JOB_NAME} ${BUILD_NUMBER}"
                }
            }
        }
    }
}
