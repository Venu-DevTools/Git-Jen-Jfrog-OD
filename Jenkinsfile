pipeline {
    agent any

    environment {
        PACKAGE_VERSION = "1.0.${BUILD_NUMBER}"
        MAVEN_HOME = tool 'Maven3'
        JFROG_CLI_HOME = tool 'jfrogcli'             // <-- your configured JFrog CLItool
        JFROG_SERVER_ID = 'jfserver'                 // <-- your configured server ID
        JFROG_REPO = 'simple-local'
        ORG_PATH = 'MyProject'
        MODULE = 'hello-world'
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
                    def jfrogCli = "${JFROG_CLI_HOME}/jf"

                    // Upload artifact
                    sh "${jfrogCli} rt upload target/${ARTIFACT_NAME} ${JFROG_REPO}/${ORG_PATH}/ --server-id=${JFROG_SERVER_ID}"

                    // Publish build info
                    sh "${jfrogCli} rt build-publish ${JFROG_CLI_BUILD_NAME} ${JFROG_CLI_BUILD_NUMBER} --server-id=${JFROG_SERVER_ID}"
                }
            }
        }
    }
}
