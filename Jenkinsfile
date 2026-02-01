pipeline { 
    agent { label 'jenkins-slave' } 

    tools { 
        jdk 'Java17' 
        maven 'Maven3' 
    } 

    environment { 
        APP_NAME = "Login-pipeline" 
        RELEASE = "1.0.0" 
        DOCKER_USER = "hariompal4" 
        IMAGE_NAME = "${DOCKER_USER}/${APP_NAME}" 
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}" 
        DOCKER_CREDS = "dockerhub-creds" 
    } 

    stages { 

        stage("Cleanup Workspace") { 
            steps { 
                cleanWs() 
            } 
        } 

        stage("Checkout from SCM") { 
            steps { 
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/Hariom-pal1/register-app' 
            } 
        } 

        stage("Build Application") { 
            steps { 
                sh "mvn clean package" 
            } 
        } 

        stage("Test Application") { 
            steps { 
                sh "mvn test" 
            } 
        } 

        stage("SonarQube Analysis") { 
            steps { 
                withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') { 
                    sh "mvn sonar:sonar" 
                } 
            } 
        } 

        stage("Quality Gate") { 
            steps { 
                waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonarqube-token' 
            } 
        } 

        stage("Build & Push Docker Image") { 
            steps { 
                script { 
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_CREDS) { 
                        def docker_image = docker.build("${IMAGE_NAME}") 
                        docker_image.push("${IMAGE_TAG}") 
                        docker_image.push("latest") 
                    } 
                } 
            } 
        } 
    } 
}
