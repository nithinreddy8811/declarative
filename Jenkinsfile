pipeline {
    agent any

    environment {
        SONARQUBE_ENV = 'MySonarQubeServer'
        NEXUS_CREDENTIALS_ID = 'nexus_server'
        TOMCAT_CREDENTIALS_ID = 'tomcat'
        NEXUS_URL = 'http://54.91.176.145:8081'
        SLACK_CHANNEL = '#jenkins-alerts'
        TOMCAT_URL = 'http://54.91.176.145:8080/manager/text'
    }

    stages {
        stage('Git Clone') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/nithinreddy8811/declarative.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    sh 'mvn clean verify sonar:sonar'
                }
            }
        }

        stage('Maven Compile') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Upload to Nexus') {
            steps {
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: "${NEXUS_URL}",
                    groupId: 'com.devops',
                    version: '1.0',
                    repository: 'hiring-app',
                    credentialsId: "${NEXUS_CREDENTIALS_ID}",
                    artifacts: [[
                        artifactId: 'declarative-app',
                        classifier: '',
                        file: 'target/*.war',
                        type: 'war'
                    ]]
                )
            }
        }

        stage('Slack Notification') {
            steps {
                slackSend(channel: "${SLACK_CHANNEL}", message: "✅ Jenkins Build #${env.BUILD_NUMBER} by nithin-a5o3317 is successful.")
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                deploy adapters: [tomcat8(credentialsId: "${TOMCAT_CREDENTIALS_ID}", path: '', url: "${TOMCAT_URL}")],
                       contextPath: 'declarative-app',
                       war: 'target/*.war'
            }
        }
    }
}
