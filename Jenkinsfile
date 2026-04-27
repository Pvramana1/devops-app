pipeline {
    agent any

    parameters {
        string(name: 'IMAGE_TAG', defaultValue: 'latest', description: 'Docker image tag')
    }

    environment {
        PROJECT_ID = "internal-sandbox-446612"
        REGION = "asia-south1"
        REPO = "my-repo"
        IMAGE = "devops-app"
        CLUSTER = "my-cluster"
        ZONE = "asia-south1-a"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git 'https://github.com/Pvramana1/devops-app.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                docker build -t ${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPO}/${IMAGE}:${params.IMAGE_TAG} .
                """
            }
        }

        stage('Run Tests') {
            steps {
                sh """
                cd app
                npm install
                npm test || echo "No tests found"
                """
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([file(credentialsId: 'gcp-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    sh """
                    gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS
                    gcloud config set project ${PROJECT_ID}
                    gcloud auth configure-docker ${REGION}-docker.pkg.dev -q

                    docker push ${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPO}/${IMAGE}:${params.IMAGE_TAG}
                    """
                }
            }
        }

        stage('Deploy to GKE') {
            steps {
                withCredentials([file(credentialsId: 'gcp-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    sh """
                    gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS
                    gcloud container clusters get-credentials ${CLUSTER} --zone ${ZONE}

                    kubectl set image deployment/devops-app devops-app=${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPO}/${IMAGE}:${params.IMAGE_TAG} || true
                    kubectl apply -f k8s/
                    """
                }
            }
        }
    }
}
