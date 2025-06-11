pipeline {
    agent any

    environment {
        SONARQUBE_SERVER = 'SonarQube'
        MAVEN_HOME = tool 'Maven'
        ARTIFACT_ID = "SimpleCustomerApp"
        GROUP_ID = "com.javatpoint"
        VERSION = "${BUILD_NUMBER}-SNAPSHOT"
    }

    stages {
        stage('Git Clone') {
            steps {
                git branch: 'feature-1.1', url: 'https://github.com/betawins/sabear_simplecutomerapp.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh "${MAVEN_HOME}/bin/mvn clean verify sonar:sonar"
                }
            }
        }

        stage('Maven Build') {
            steps {
                sh "${MAVEN_HOME}/bin/mvn clean package"
            }
        }

        stage('Publish to Nexus') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'nexus_server', usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
                    script {
                        def warFile = "target/${env.ARTIFACT_ID}-${env.VERSION}.war"
                        def nexusPath = "${env.GROUP_ID.replace('.', '/')}/${env.ARTIFACT_ID}/${env.VERSION}/${env.ARTIFACT_ID}-${env.VERSION}.war"
                        def uploadUrl = "http://54.91.176.145:8081/repository/hiring-app/${nexusPath}"

                        sh """
                            echo "Uploading ${warFile} to Nexus..."
                            curl -v -u $NEXUS_USER:$NEXUS_PASS --upload-file ${warFile} ${uploadUrl}
                        """
                    }
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'tomcat', usernameVariable: 'TOMCAT_USER', passwordVariable: 'TOMCAT_PASS')]) {
                    script {
                        def warFile = "target/${env.ARTIFACT_ID}-${env.VERSION}.war"
                        sh """
                            curl -u $TOMCAT_USER:$TOMCAT_PASS --upload-file ${warFile} http://54.91.176.145:8080/manager/text/deploy?path=/sabearapp&update=true
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            slackSend (
                channel: '#jenkins-alerts',
                color: 'good',
                message: "✅ Build SUCCESSFUL for ${env.JOB_NAME} #${env.BUILD_NUMBER}"
            )
        }
        failure {
            slackSend (
                channel: '#jenkins-alerts',
                color: '#ff0000',
                message: "❌ Build FAILED for ${env.JOB_NAME} #${env.BUILD_NUMBER}"
            )
        }
    }
}
