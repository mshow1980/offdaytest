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
    }
}