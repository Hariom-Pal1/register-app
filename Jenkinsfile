pipeline {
    agent { label 'jenkins-slave' }
    tools {
        jdk 'Java17'
        maven 'Maven3'
    }
    environment {
        APP_NAME = "register-app-pipeline"
        RELEASE = "1.0.0"
        DOCKER_USER = "hariompal4"
        DOCKER_PASS = 'dockerhub' 
        IMAGE_NAME = "${DOCKER_USER}/${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
        JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN") 
    }
    stages {
        stage("Cleanup Workspace") {
            steps { cleanWs() }
        }

        stage("Checkout from SCM") {
            steps {
                // Aapka Github URL
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/Hariom-Pal1/register-app.git'
            }
        }

        stage("Build & Test") {
            steps { sh "mvn clean package test" }
        }

        stage("SonarQube Analysis") {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') { 
                        sh "mvn sonar:sonar"
                    }
                }
            }
        }

        stage("Build & Push Docker Image") {
            steps {
                script {
                    docker.withRegistry('', DOCKER_PASS) {
                        def docker_image = docker.build("${IMAGE_NAME}")
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
                }
            }
        }

        stage("Trivy Scan") {
            steps {
                sh "docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image ${IMAGE_NAME}:latest --severity HIGH,CRITICAL"
            }
        }

        stage("Trigger CD Pipeline") {
            steps {
                // Curl ki jagah ye use karke dekho, ye fail nahi hoga
                build job: 'gitops-register-app-cd', 
                parameters: [string(name: 'IMAGE_TAG', value: "${IMAGE_TAG}")],
                wait: false
            }
        }
    }
}

