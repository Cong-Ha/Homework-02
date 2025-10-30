pipeline {
    agent any
    
    environment {
        NODE_VERSION = '18'
        DOCKER_IMAGE_NAME = 'uzomaki/homework-02'
        DOCKER_TAG = "${env.BUILD_NUMBER}"
        DOCKERHUB_CREDENTIALS_ID = 'docker_id'
        DOCKERHUB_CREDENTIALS = credentials('docker_id')
        DOCKERHUB_URL = 'https://registry.hub.docker.com'
    }
    
    stages {
        stage('Clone Repository') {
            steps {
                checkout scm
            }
        }

        stage('Static Security Scan') {
            agent {label 'App-Server'}
            steps {
                script {
                    snykSecurity(
                        snykInstallation: 'Snyk',
                        snykTokenId: 'snyk_token',
                        severity: 'critical'
                    )
                }
            }
        }

        stage('SonarQube Analysis') {
            agent {label 'App-Server'}
            steps {
                script {
                    def scannerHome = tool 'SonarQube-Scanner'
                    withSonarQubeEnv('sonarqube') {
                        sh "${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=nodejs-chatapp \
                            -Dsonar.sources=."
                    }
                }
            }
        }

        stage('BUILD-AND-TAG') {
            agent {label 'App-Server'}
            steps {
                script {
                    echo "Building Docker image ${DOCKER_IMAGE_NAME}..."
                    app = docker.build(DOCKER_IMAGE_NAME)
                    app.tag("latest")
                }
            }
        }
        
        stage('Push Docker Image to Docker Hub') {
            agent {label 'App-Server'}
            steps {
                script {
                    echo "Pushing Docker image ${DOCKER_IMAGE_NAME}:latest to Docker Hub..."
                    docker.withRegistry(env.DOCKERHUB_URL, "${env.DOCKERHUB_CREDENTIALS_ID}") {
                        app.push("latest")
                        app.push("${env.BUILD_NUMBER}")
                    }
                }
            }
        }
        
        stage('Deployment') {
            agent {label 'App-Server'}
            steps{
                echo "Starting deployment using docker-compose..."
                script{
                    dir("$WORKSPACE") {
                        sh '''
                            docker-compose down
                            docker-compose up -d
                            docker ps
                        '''    
                    }
                }
                echo 'Deployment completed successfully'
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline succeeded! NodeJS Chat Application is running on port 3001'
        }
        failure {
            echo 'Pipeline failed!'
        }
        always {
            echo 'Pipeline execution completed'
        }
    }
}

