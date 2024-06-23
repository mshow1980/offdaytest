pipeline {
    agent any 
    tools {
        jdk 'jdk17'
        maven 'mvn3'
    }
    environment {
        APP_NAME = "verified-sunday"
        RELEASE = "1.2.0"
        DOCKER_USER = "mshow1980"
        REGISTRY_CREDS = "docker-login"
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
        JENKINS_API_TOKEN = credentials ('JENKINS_API_TOKEN')
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
        stage ('Quality Analysis') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'SOnar-login'
                }
            }
        }
        stage ('Docker Build') {
            steps {
                script {
                withDockerRegistry(credentialsId: 'docker-login', toolName: 'Docker') {
                    docker_image = docker.build "${IMAGE_NAME}"
                    }
                withDockerRegistry(credentialsId: 'docker-login', toolName: 'Docker') {
                    docker_image.push("${BUILD_NUMBER}")
                    docker_image.push('latest')
                    docker_image.push("${IMAGE_TAG}")
                    }
                }
            }
        }
        stage ('Trivy Image Scan'){
            steps {
                script {
                    sh " trivy image ${IMAGE_NAME}  > ImageScanResult.txt "
                }
            }
        }
        stage ('Deleting Images & docker logout') {
            steps {
                script {
                    sh """
                    docker rmi ${IMAGE_NAME}:latest
                    docker rmi ${IMAGE_NAME}:${IMAGE_TAG}
                    docker logout
                    """
                }
            }
        }
        stage ('Updating Manifest') {
            steps {
                script {
                sh "curl -v -X POST http://54.237.55.4:8080/second-job/buildWithParameters?IMAGE_TAG=${IMAGE_TAG} --user Scion_Scope:${JENKINS_API_TOKEN}"
                }
            }
        }
    }
}