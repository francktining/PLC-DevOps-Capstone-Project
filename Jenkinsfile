pipeline {
    agent any

    tools {
        // SonarScanner installed via Jenkins Global Tool Configuration
        sonarScanner 'sonar-scanner'
    }

    environment {
        // Docker Hub
        DOCKER_HUB_CREDENTIALS = 'dockerhub-creds'
        DOCKER_HUB_USERNAME = 'francktining'
        IMAGE_NAME = "flask-app"
        IMAGE_TAG = "latest"

        // SonarQube
        SONARQUBE_SERVER = 'SonarQube'   
        SONAR_PROJECT_KEY = 'PLC-project'
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/francktining/PLC-DevOps-Capstone-Project.git',
                    credentialsId: 'github-creds'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh """
                    sonar-scanner \
                      -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                      -Dsonar.sources=.
                    """
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_HUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_HUB_CREDENTIALS) {
                        docker.image("${DOCKER_HUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}").push()
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ SUCCESS: Image pushed -> ${DOCKER_HUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}"
        }
        failure {
            echo "❌ Pipeline failed!"
        }
    }
}
