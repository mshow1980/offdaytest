pipeline {
    agent any 
    tools {
        jdk 'jdk17'
        maven 'mvn3'
    }
    environment {
        APP_NAME = "offdaytest"
        RELEASE = "1.2.0"
        DOCKER_USER = "mshow1980"
        REGISTRY_CREDS = "docker-login"
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
    }
    stages {
        stage ('CleanWS') {
            steps {
                cleanWs()
            }
        }
        stage ('CheckOut SCM') {
            steps {
                checkout scmGit(branches: [[name: '*/main']], 
                extensions: [], userRemoteConfigs: [[url: 'https://github.com/mshow1980/offdaytest.git']])
            }
        }
        stage ('Installing Maven Packages'){
            steps {
                script{
                    sh 'mvn clean install'
                    }
                }
            }
        stage ('Testing Application') {
            steps{
                script {
                    sh 'mvn test'
                }
            }
        }
        stage ('OWASP Dependecy Checks'){
            steps {
                script{
                dependencyCheck additionalArguments: ''' 
                    -o "./" 
                    -s "./"
                    -f "ALL" 
                    --prettyPrint''', odcInstallation: 'OWASP-DC'
                dependencyCheckPublisher pattern: 'dependency-check-report.xml'
                }
            }
        }
        stage ('Trivy FS Scan'){
            steps {
                script {
                    sh " trivy fs . > scanresult.txt"
                }
            }
        }
        stage ('SOanrqube Analysis'){
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'SOnar-login') {
                        sh ' mvn sonar:sonar'
                    }
                }
            }
        }
    }
}