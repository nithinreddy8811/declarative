pipeline {
    agent any

    environment {
        SONAR_PROJECT_KEY = "hiring-ap"
        SONAR_HOST_URL = "http://100.25.214.16:9000"
        SONAR_AUTH_TOKEN = credentials('sonarqube')

        NEXUS_URL = "http://54.91.176.145:8081"
        NEXUS_REPO = "hiring-app"
        NEXUS_CREDENTIAL_ID = "nexus_server"

        TOMCAT_URL = "http://54.91.176.145:8080/manager/text"
        TOMCAT_CREDENTIAL_ID = "tomcat"
        WAR_FILE = "target/SimpleCustomerApp.war"
    }

    stages {
        stage('Git Clone') {
            steps {
                git branch: 'master', url: 'https://github.com/nithinreddy8811/declarative.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh """
                        mvn clean verify sonar:sonar \
                          -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                          -Dsonar.host.url=${SONAR_HOST_URL} \
                          -Dsonar.login=${SONAR_AUTH_TOKEN}
                    """
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
                script {
                    def artifact = "target/SimpleCustomerApp.war"
                    def artifactName = "SimpleCustomerApp.war"
                    def groupId = "com.customer"
                    def artifactId = "SimpleCustomerApp"
                    def version = "1.0.0"
                    def repository = "${NEXUS_REPO}"

                    sh """
                        curl -v -u ${NEXUS_CREDENTIAL_ID} --upload-file ${artifact} \
                        ${NEXUS_URL}/repository/${repository}/${groupId.replace('.', '/')}/${artifactId}/${version}/${artifactName}
                    """
                }
            }
        }

        stage('Slack Notification') {
            steps {
                slackSend(channel: '#jenkins-alerts', message: "Build Successful: ${env.JOB_NAME} - ${env.BUILD_NUMBER}", color: '#00FF00')
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                script {
                    def war = "target/SimpleCustomerApp.war"
                    sh """
                        curl -u ${TOMCAT_CREDENTIAL_ID} --upload-file ${war} "${TOMCAT_URL}/deploy?path=/SimpleCustomerApp&update=true"
                    """
                }
            }
        }
    }

    post {
        failure {
            slackSend(channel: '#jenkins-alerts', message: "Build Failed: ${env.JOB_NAME} - ${env.BUILD_NUMBER}", color: '#FF0000')
        }
    }
}
