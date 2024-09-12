pipeline {

    agent any

    tools {
        nodejs 'node22'
    }

    environment {
        DOCKER_CREDENTIALS = credentials('f81abbea-2b04-4323-9b98-5964dfd2af75')
        IMAGE_TAG = "v.0.0.${env.BUILD_NUMBER}"
        IMAGE_NAME = "idrisniyi94/netflix-demo:${IMAGE_TAG}"
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from Git') {
            steps {
                git branch: 'master', url: 'https://github.com/stwins60/Netflix-Project.git'
            }
        }
        stage("Sonarqube Analysis") {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=netflix-app -Dsonar.projectName=netflix-app"
                }
            }
        }
        stage("Quality Gate"){
            steps {
                withSonarQubeEnv('sonar-server') {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }
        stage("Install Dependency") {
            steps {
                sh "npm install"
            }
        }
        stage('OWASP Vulnerability Scanner') {
            steps {
                script {
                    dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                    dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
                }
            }
        }
    }
}