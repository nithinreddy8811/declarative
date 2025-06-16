pipeline {
    agent any

    tools {
        maven 'Maven-3.9.4'
        jdk 'java21'
    }

    parameters {
        booleanParam(name: 'RUN_SONARQUBE', defaultValue: true, description: 'Run SonarQube analysis')
        booleanParam(name: 'RUN_MAVEN_DEPLOY', defaultValue: true, description: 'Run Maven build and deploy to Nexus')
        booleanParam(name: 'DEPLOY_TO_TOMCAT', defaultValue: true, description: 'Deploy WAR to Tomcat')
    }

    environment {
        NEXUS_REPO_URL = 'http://54.227.50.97:8081/repository/simplecustomerapp/'
    }

    stages {
        stage('Git Clone') {
            steps {
                git branch: 'master', url: 'https://github.com/nithinreddy8811/declarative.git'
            }
        }

        stage('SonarQube Analysis') {
            when {
                expression { return params.RUN_SONARQUBE }
            }
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn clean verify sonar:sonar'
                }
            }
        }

        stage('Maven Build & Deploy to Nexus') {
            when {
                expression { return params.RUN_MAVEN_DEPLOY }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'nexus_server', usernameVariable: 'NEXUS_USERNAME', passwordVariable: 'NEXUS_PASSWORD')]) {
                    sh """
                        mvn clean deploy -DskipTests \
                        -DaltDeploymentRepository=simplecustomerapp::default::http://$NEXUS_USERNAME:$NEXUS_PASSWORD@54.227.50.97:8081/repository/simplecustomerapp/
                    """
                }
            }
        }

        stage('Deploy to Tomcat') {
            when {
                expression { return params.DEPLOY_TO_TOMCAT }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'tomcat', usernameVariable: 'TOMCAT_USERNAME', passwordVariable: 'TOMCAT_PASSWORD')]) {
                    sh """
                        WAR_FILE=\$(ls target/*.war | head -n 1)
                        cp \$WAR_FILE target/customerapp.war

                        curl -v -u ${TOMCAT_USERNAME}:${TOMCAT_PASSWORD} \
                        --upload-file target/customerapp.war \
                        "http://54.227.50.97:8080/manager/text/deploy?path=/customerapp&update=true"
                    """
                }
            }
        }

        stage('Slack Notification') {
            steps {
                slackSend(
                    color: '#00FF00',
                    message: "✅ Build Successful: ${env.JOB_NAME} #${env.BUILD_NUMBER} → <${env.BUILD_URL}|View Logs>",
                    tokenCredentialId: 'slack-token'
                )
            }
        }
    }
}
