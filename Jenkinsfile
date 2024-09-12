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
        TMDB_API_KEY = credentials("dc60bb2d-7c4e-4128-95d1-61e72b330993")
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
                    dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit --nvdApiKey 24d49913-f86d-4c46-a43c-49388a3383ef', odcInstallation: 'DP-Check'
                    dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
                }
            }
        }
        stage("Docker login") {
            steps {
                script {
                    sh "echo $DOCKER_CREDENTIALS_PSW | docker login -u $DOCKER_CREDENTIALS_USR --password-stdin"
                }
            }
        }
        stage("Docker Build") {
            steps {
                script {
                    sh "docker build -t $IMAGE_NAME --build-arg TMDB_V3_API_KEY=$TMDB_API_KEY ."
                }
            }
        }
        stage("Trivy Image Scan") {
            steps {
                script {
                    sh "trivy image $IMAGE_NAME"
                }
            }
        }
        stage("Docker Push") {
            steps {
                script {
                    sh "docker push $IMAGE_NAME"
                }
            }
        }
    }
}