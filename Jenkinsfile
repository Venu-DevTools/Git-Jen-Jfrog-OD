pipeline {
    agent any

    environment {
        PACKAGE_VERSION = "1.0.${BUILD_NUMBER}"
        MAVEN_HOME = tool 'Maven3'
        JFROG_CLI_HOME = tool 'jfrogcli'             // Jenkins > Tools > JFrog CLI
        JFROG_SERVER_ID = 'jfserver'                 // Jenkins > Manage > System > JFrog Server ID
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
                withCredentials([usernamePassword(credentialsId: 'JFR', usernameVariable: 'JF_USER', passwordVariable: 'JF_PASS')]) {
                    script {
                        def jfrogCli = "${JFROG_CLI_HOME}/jf"

                        // Configure JFrog CLI with credentials (basic-auth-only to avoid encryption error)
                        sh '''
                            echo y | ${jfrogCli} config add ${JFROG_SERVER_ID} \
                                --url=http://130.131.164.192:8082/artifactory \
                                --user=$JF_USER \
                                --password=$JF_PASS \
                                --interactive=false \
                                --basic-auth-only
                        '''

                        // Upload the JAR
                        sh "${jfrogCli} rt upload target/${ARTIFACT_NAME} ${JFROG_REPO}/${ORG_PATH}/ --server-id=${JFROG_SERVER_ID}"

                        // Publish build info
                        sh "${jfrogCli} rt build-publish ${JFROG_CLI_BUILD_NAME} ${JFROG_CLI_BUILD_NUMBER} --server-id=${JFROG_SERVER_ID}"
                    }
                }
            }
        }
    }
}
