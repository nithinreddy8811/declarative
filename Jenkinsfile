pipeline {
    agent any

    tools {
        maven 'MVN_HOME'
        jdk 'java21'
    }

    environment {
        SONARQUBE_SERVER = 'SonarQube'
    }

    stages {
        stage('Git Clone') {
            steps {
                git branch: 'feature-1.1', url: 'https://github.com/nithinreddy8811/declarative.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn clean verify sonar:sonar'
                }
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Publish to Nexus') {
            steps {
                script {
                    def pom = readMavenPom file: 'pom.xml'
                    def artifactId = pom.artifactId
                    def groupId = pom.groupId
                    def version = pom.version

                    withCredentials([usernamePassword(credentialsId: 'nexus_server', usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
                        def warFile = "target/${artifactId}-${version}.war"
                        def nexusPath = "${groupId.replace('.', '/')}/${artifactId}/${version}/${artifactId}-${version}.war"
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
                script {
                    def pom = readMavenPom file: 'pom.xml'
                    def artifactId = pom.artifactId
                    def version = pom.version
                    def warFile = "target/${artifactId}-${version}.war"

                    withCredentials([usernamePassword(credentialsId: 'tomcat', usernameVariable: 'TOMCAT_USER', passwordVariable: 'TOMCAT_PASS')]) {
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
