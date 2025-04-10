pipeline {
    agent any

    environment {
        PACKAGE_VERSION = "1.0.${BUILD_NUMBER}"
        MAVEN_HOME = tool 'Maven3'
        JFROG_CLI_HOME = tool 'JFrogCLI'
        JFROG_SERVER_ID = 'jfrogserver'
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

        stage('Configure JFrog CLI') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'jfrog-creds', usernameVariable: 'JFROG_USER', passwordVariable: 'JFROG_PASS')]) {
                    sh '''
                        jf config add jfrogserver \
                            --url=http://130.131.164.192:8082 \
                            --user=$JFROG_USER \
                            --password=$JFROG_PASS \
                            --interactive=false
                    '''
                }
            }
        }

        stage('Upload to JFrog Artifactory') {
            steps {
                script {
                    def jfrogCli = "${JFROG_CLI_HOME}/jf"

                    sh "${jfrogCli} rt upload --server-id=${JFROG_SERVER_ID} target/${ARTIFACT_NAME} ${JFROG_REPO}/${ORG_PATH}/${MODULE}-${PACKAGE_VERSION}.jar"

                    sh "${jfrogCli} rt build-publish --server-id=${JFROG_SERVER_ID} ${JFROG_CLI_BUILD_NAME} ${JFROG_CLI_BUILD_NUMBER}"
                }
            }
        }
    }
}
