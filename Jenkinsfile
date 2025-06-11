pipeline {
    agent any

    tools {
        maven 'Maven3.9.4'  // Must match Maven name in Jenkins Global Tools
    }

    environment {
        SONARQUBE_ENV = 'MySonarQubeServer' // Set this in Jenkins > Manage Jenkins > Configure System
        NEXUS_URL = 'http://54.91.176.145:8081/repository/hiring-app/'
        NEXUS_CREDENTIAL_ID = 'nexus_server'
        TOMCAT_URL = 'http://54.91.176.145:8080/manager/text'
        TOMCAT_CREDENTIAL_ID = 'tomcat'
        SLACK_CHANNEL = '#jenkins-alerts'
        GIT_BRANCH = 'feature-1.1'
    }

    stages {
        stage('Git Clone') {
            steps {
                git branch: "${env.GIT_BRANCH}",
                    url: 'https://github.com/nithinreddy8811/declarative.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${env.SONARQUBE_ENV}") {
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
                withCredentials([usernamePassword(credentialsId: "${env.NEXUS_CREDENTIAL_ID}", usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
                    sh '''
                        curl -v -u $NEXUS_USER:$NEXUS_PASS --upload-file target/*.war ${NEXUS_URL}
                    '''
                }
            }
        }

        stage('Slack Notification') {
            steps {
                slackSend (
                    channel: "${env.SLACK_CHANNEL}",
                    message: "✅ Build Successful for ${env.JOB_NAME} - ${env.BUILD_NUMBER}",
                    color: '#36a64f'
                )
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${env.TOMCAT_CREDENTIAL_ID}", usernameVariable: 'TOMCAT_USER', passwordVariable: 'TOMCAT_PASS')]) {
                    sh '''
                        curl -v --upload-file target/*.war "http://${TOMCAT_USER}:${TOMCAT_PASS}@54.91.176.145:8080/manager/text/deploy?path=/simpleapp&update=true"
                    '''
                }
            }
        }
    }

    post {
        failure {
            slackSend (
                channel: "${env.SLACK_CHANNEL}",
                message: "❌ Build Failed for ${env.JOB_NAME} - ${env.BUILD_NUMBER}",
                color: '#ff0000'
            )
        }
    }
}
