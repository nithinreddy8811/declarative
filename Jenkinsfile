pipeline {
    agent any

    environment {
        MAVEN_HOME = "/opt/apache-maven-3.9.4/bin"
        SONARQUBE_ENV = 'SonarQube' // must match name in Jenkins > Configure System > SonarQube section
    }

    stages {
        stage('Git Clone') {
            steps {
                git branch: 'feature-1.1', url: 'https://github.com/nithinreddy8811/declarative.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    sh "${MAVEN_HOME}/mvn clean verify sonar:sonar -Dsonar.projectKey=hiring-ap -Dsonar.branch.name=main"
                }
            }
        }

        stage('Maven Compile') {
            steps {
                sh "${MAVEN_HOME}/mvn clean compile"
            }
        }

        stage('Upload to Nexus') {
            steps {
                sh """
                    ${MAVEN_HOME}/mvn deploy \
                    -DaltDeploymentRepository=hiring-app::default::http://54.91.176.145:8081/repository/hiring-app/ \
                    -DskipTests
                """
            }
        }

        stage('Slack Notification') {
            steps {
                slackSend(
                    channel: '#jenkins-alerts',
                    color: 'good',
                    message: "Build SUCCESSFUL for Job: ${env.JOB_NAME} (Build #${env.BUILD_NUMBER}) by ${env.BUILD_USER}"
                )
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                script {
                    def warFile = findFiles(glob: '**/target/*.war')[0].path
                    withCredentials([usernamePassword(credentialsId: 'tomcat', usernameVariable: 'TOMCAT_USER', passwordVariable: 'TOMCAT_PASS')]) {
                        sh """
                            curl -u $TOMCAT_USER:$TOMCAT_PASS \
                            --upload-file ${warFile} \
                            http://54.91.176.145:8080/manager/text/deploy?path=/hiringapp&update=true
                        """
                    }
                }
            }
        }
    }

    post {
        failure {
            slackSend(
                channel: '#jenkins-alerts',
                color: 'danger',
                message: "Build FAILED for Job: ${env.JOB_NAME} (Build #${env.BUILD_NUMBER}) by ${env.BUILD_USER}"
            )
        }
    }
}
