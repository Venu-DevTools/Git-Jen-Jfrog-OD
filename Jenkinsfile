pipeline {
    agent any

    environment {
        PACKAGE_VERSION = "1.0.${BUILD_NUMBER}"
        MAVEN_HOME = tool 'Maven3'
    }

    stages {
        stage('Checkout') {
            steps {
                git credentialsId: 'github-creds', url: 'https://github.com/Venu-DevTools/Git-Jen-Jfrog-OD.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh "${MAVEN_HOME}/bin/mvn clean package -Djar.finalName=hello-world.${PACKAGE_VERSION}"
            }
        }

        stage('Archive JAR') {
            steps {
                archiveArtifacts artifacts: "target/hello-world.${PACKAGE_VERSION}.jar", fingerprint: true
            }
        }

        stage('Upload to JFrog Artifactory') {
            steps {
                script {
                    def server = Artifactory.server 'jfrog-server' // Use your configured Server ID
                    def uploadSpec = """{
                        "files": [
                            {
                                "pattern": "target/hello-world.${PACKAGE_VERSION}.jar",
                                "target": "simple-local/MyProject/hello-world-${PACKAGE_VERSION}.jar"
                            }
                        ]
                    }"""
                    server.upload(spec: uploadSpec)
                }
            }
        }
    }
}
