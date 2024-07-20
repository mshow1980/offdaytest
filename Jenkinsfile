pipeline {
    agent any 
    environment {
        APP_NAME = "verified-saturday"
        RELEASE = "1.2.0"
        DOCKER_USER = "mshow1980"
        REGISTRY_CREDS = "docker-login"
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
        GIT_USER_NAME = "mshow1980"
        GIT_REPO_NAME = "offdaytest"
    }
    stages{
        stage ('CleanWS') {
            steps {
                script {
                    cleanWs()
                }
            }
        }
        stage ('Checkout SCM') {
            steps {
                script {
                    checkout scmGit(branches: [[name: '*deploynent']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/mshow1980/offdaytest.git']])
                }
            }
        }
        stage ('Updating manifest') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'GITHUB_TOKEN', variable: 'GITHUB']) {
                    sh 
                    """
                        git config  user.name "mshow1980"
                        git config  user.email "mshow1980@aol.com"
                        git switch deployment
                        cat deployment.yaml
                        sed -i 's/${APP_NAME}.*/${APP_NAME}:${IMAGE_TAG}/g' deployment.yaml
                        cat deployment.yaml
                        git add deployment.yaml
                        git commit -m 'Updated the deployment file'
                        git push origin deployment
                        git push https://${GITHUB}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:deployment
                        """
                    }
                }
            }
        }
    }
}
