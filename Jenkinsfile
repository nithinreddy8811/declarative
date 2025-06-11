pipeline {
    agent any

    tools {
        maven 'MVN_HOME'
        jdk 'java21'
    }

    environment {
        // Nexus Config
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "54.91.176.145:8081"
        NEXUS_REPOSITORY = "hiring-app"
        NEXUS_CREDENTIAL_ID = "nexus_server"

        // SonarQube
        SONARQUBE_URL = "http://100.25.214.16:9000"
        SONARQUBE_CREDENTIAL_ID = "sonarqube"

        // Tomcat
        TOMCAT_URL = "http://54.91.176.145:8080/manager/html"
        TOMCAT_CREDENTIAL_ID = "tomcat"

        // Slack
        SLACK_CHANNEL = "#jenkins-alerts"
    }

    stages {
        stage('Clone Code') {
            steps {
                git branch: 'master', url: 'https://github.com/nithinreddy8811/declarative.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn -Dmaven.test.failure.ignore=true clean install'
            }
        }

        stage('Run SonarQube Scan') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    withCredentials([string(credentialsId: "${SONARQUBE_CREDENTIAL_ID}", variable: 'SONAR_TOKEN')]) {
                        sh """
                            mvn sonar:sonar \
                                -Dsonar.projectKey=sabear_simplecutomerapp \
                                -Dsonar.host.url=$SONARQUBE_URL \
                                -Dsonar.login=$SONAR_TOKEN
                        """
                    }
                }
            }
        }

        stage('Publish to Nexus') {
            steps {
                script {
                    def pom = readMavenPom file: 'pom.xml'
                    def warFile = "target/${pom.artifactId}-${pom.version}.${pom.packaging}"

                    if (!fileExists(warFile)) {
                        error "WAR file not found at ${warFile}"
                    }

                    nexusArtifactUploader(
                        nexusVersion: NEXUS_VERSION,
                        protocol: NEXUS_PROTOCOL,
                        nexusUrl: NEXUS_URL,
                        repository: NEXUS_REPOSITORY,
                        credentialsId: NEXUS_CREDENTIAL_ID,
                        groupId: pom.groupId,
                        version: pom.version,
                        artifacts: [
                            [artifactId: pom.artifactId, classifier: '', file: warFile, type: pom.packaging],
                            [artifactId: pom.artifactId, classifier: '', file: 'pom.xml', type: 'pom']
                        ]
                    )
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                script {
                    def pom = readMavenPom file: 'pom.xml'
                    def warFile = "target/${pom.artifactId}-${pom.version}.${pom.packaging}"

                    deploy adapters: [
                        tomcat9(
                            credentialsId: TOMCAT_CREDENTIAL_ID,
                            path: '',
                            url: TOMCAT_URL
                        )
                    ],
                    contextPath: "/${pom.artifactId}",
                    war: warFile
                }
            }
        }
    }

    post {
        success {
            slackSend(channel: "${SLACK_CHANNEL}", message: "✅ *SUCCESS*: Build & Deploy for `sabear_simplecutomerapp` on Jenkins. See: ${env.BUILD_URL}", color: "#36a64f")
        }
        failure {
            slackSend(channel: "${SLACK_CHANNEL}", message: "❌ *FAILURE*: Build & Deploy for `sabear_simplecutomerapp` on Jenkins. See: ${env.BUILD_URL}", color: "#ff0000")
        }
    }
}
