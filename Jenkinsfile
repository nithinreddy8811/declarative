pipeline {
    agent any

    tools {
        maven 'M3' // Must match the name in Jenkins > Global Tool Configuration
    }

    environment {
        SONAR_PROJECT_KEY = 'hiring-ap'
        SONAR_HOST_URL = 'http://100.25.214.16:9000'
        NEXUS_REPO = 'http://54.91.176.145:8081/repository/hiring-app/'
        NEXUS_CREDENTIAL_ID = 'nexus_server'
        TOMCAT_URL = 'http://54.91.176.145:8080/manager/text'
        TOMCAT_CREDENTIAL_ID = 'tomcat'
        ARTIFACT_ID = 'SimpleCustomerApp'
    }

    stages {
        stage('Git Clone') {
            steps {
                git branch: 'master', url: 'https://github.com/nithinreddy8811/declarative.git'
            }
        }

        stage('SonarQube Analysis') {
            environment {
                SONAR_AUTH_TOKEN = credentials('sonarqube')
            }
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh """
                        mvn clean verify sonar:sonar \
                          -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                          -Dsonar.host.url=${SONAR_HOST_URL}
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
                    def version = sh(
                        script: "mvn help:evaluate -Dexpression=project.version -q -DforceStdout",
                        returnStdout: true
                    ).trim()

                    def fileName = "target/${ARTIFACT_ID}-${version}.war"

                    withCredentials([usernamePassword(credentialsId: NEXUS_CREDENTIAL_ID, usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
                        sh """
                            curl -v -u $NEXUS_USER:$NEXUS_PASS --upload-file ${fileName} ${NEXUS_REPO}
                        """
                    }
                }
            }
        }

        stage('Slack Notification') {
            steps {
                echo 'Slack integration can be added here.'
                // slackSend(...) if Slack is configured
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                script {
                    def version = sh(
                        script: "mvn help:evaluate -Dexpression=project.version -q -DforceStdout",
                        returnStdout: true
                    ).trim()

                    def fileName = "${ARTIFACT_ID}-${version}.war"

                    withCredentials([usernamePassword(credentialsId: TOMCAT_CREDENTIAL_ID, usernameVariable: 'TOMCAT_USER', passwordVariable: 'TOMCAT_PASS')]) {
                        sh """
                            curl -v -u $TOMCAT_USER:$TOMCAT_PASS -T target/${fileName} ${TOMCAT_URL}/deploy?path=/customerapp&update=true
                        """
                    }
                }
            }
        }
    }

    post {
        failure {
            echo 'Build failed!'
            // Add Slack or email notification if required
        }
        success {
            echo 'Build and Deployment successful!'
        }
    }
}
