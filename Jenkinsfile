pipeline {
    agent any  // Use the 'any' agent, similar to the working App Engine pipeline

    environment {
        PROJECT_ID = 'swiggy-project-453308'  // GCP Project ID
        GOOGLE_APPLICATION_CREDENTIALS = credentials('gcp-service-account')  // Service account credentials
        DOCKER_HUB_CREDENTIALS_USR = 'divya578'  // Your Docker Hub username
        IMAGE_NAME = 'cloudrun'  // Docker image name
        DOCKER_HUB_CREDENTIALS_PSWD = credentials('docker-hub-password')  // Docker Hub password credentials
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/Divya555555/jenkinspipeline.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image
                    sh "docker build -t ${DOCKER_HUB_CREDENTIALS_USR}/${IMAGE_NAME}:${BUILD_NUMBER} ."
                }
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                script {
                    // Login to Docker Hub using stored credentials
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-password', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh "echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin"
                    }
                    // Push the Docker image to Docker Hub
                    sh "docker push ${DOCKER_HUB_CREDENTIALS_USR}/${IMAGE_NAME}:${BUILD_NUMBER}"
                }
            }
        }

        stage('Deploy to Google Cloud Run') {
            steps {
                script {
                    withCredentials([file(credentialsId: 'gcp-service-account', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                        // Authenticate using service account key
                        sh "gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS"
                        sh "gcloud config set project ${PROJECT_ID}"

                        // Deploy to Cloud Run
                        sh "gcloud run deploy ${IMAGE_NAME} \
                            --image docker.io/${DOCKER_HUB_CREDENTIALS_USR}/${IMAGE_NAME}:${BUILD_NUMBER} \
                            --platform managed \
                            --region us-central1 \
                            --allow-unauthenticated"

                        // Make service publicly accessible
                        sh "gcloud run services add-iam-policy-binding ${IMAGE_NAME} \
                            --region us-central1 \
                            --member='allUsers' \
                            --role='roles/run.invoker'"
                    }
                }
            }
        }

        stage('Cleanup Workspace') {
            steps {
                script {
                    cleanWs()
                }
            }
        }
    }
}
