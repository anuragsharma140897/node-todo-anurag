pipeline {
    agent any
    environment {
        DOCKERHUB_CREDENTIALS = 'DockerHubCreds'
        DOCKER_IMAGE_NAME = 'anuragsharma97/node-todo'
        DOCKER_IMAGE_TAG = 'latest'
        SONAR_HOME = tool 'SonarScanner' // Assuming 'SonarScanner' is the configured tool name for SonarQube Scanner
    }
    stages {
        stage('Git Clone') {
            steps {
                git url: 'https://github.com/anuragsharma140897/node-todo-anurag.git', branch: 'main'
            }
        }
        stage('Sonarqube Analysis') {  
            steps {
                withSonarQubeEnv('SonarServer'){   
                    sh "$SONAR_HOME/bin/sonar-scanner -Dsonar.projectName='nodetodo' -Dsonar.projectKey='node-odo'"
                }
            }
        }
        stage('OWASP') {
            steps {
                dependencyCheck additionalArguments: '--scan', odcInstallation: 'OWASP'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t node-todo:latest .'
                echo 'Code built successfully'
            }
        }
        stage('Trivy') {
            steps {
                sh 'trivy image node-todo:latest'
            }
        }
        stage('Deploy Docker Container') {
            steps {
                sh 'docker-compose up -d' 
                echo 'Deployed successfully'
            }
        }
        
        stage('Push to Private Docker Repo') {
            steps {
                script {
                    // Push the Docker image to Docker Hub
                    sh 'docker tag node-todo:latest anuragsharma97/node-todo:latest'
                    withDockerRegistry([credentialsId: DOCKERHUB_CREDENTIALS, url: "https://index.docker.io/v1/"]) {
                        docker.image("$DOCKER_IMAGE_NAME:$DOCKER_IMAGE_TAG").push()
                    }
                }
            }
        }
        
        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'MINUTES') {
                script {
                def qualityGate = waitForQualityGate abortPipeline: false
                if (qualityGate.status != 'OK') {
                    echo "Quality Gate check failed: ${qualityGate.status}"
                    echo "Quality Gate error message: ${qualityGate.error}"
                }
            }
        }
    }
}

        
    }
}

