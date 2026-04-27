pipeline {
    agent any

    environment {
        PROJECT_ID = "internal-sandbox-446612"
        REGION = "asia-south1"
        REPO = "my-repo"          // ← replace if your repo name is different
        IMAGE = "devops-app"
        CLUSTER = "my-cluster"    // ← replace with your actual cluster name
        ZONE = "asia-south1-a"
    }

    stages {

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t $REGION-docker.pkg.dev/$PROJECT_ID/$REPO/$IMAGE:latest .
                '''
            }
        }

        stage('Push Image to Artifact Registry') {
            steps {
                withCredentials([file(credentialsId: 'gcp-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    sh '''
                    gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS
                    gcloud config set project $PROJECT_ID
                    gcloud auth configure-docker $REGION-docker.pkg.dev -q

                    docker push $REGION-docker.pkg.dev/$PROJECT_ID/$REPO/$IMAGE:latest
                    '''
                }
            }
        }

        stage('Deploy to GKE') {
            steps {
                withCredentials([file(credentialsId: 'gcp-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    sh '''
                    gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS
                    gcloud container clusters get-credentials $CLUSTER --zone $ZONE

                    kubectl set image deployment/devops-app devops-app=$REGION-docker.pkg.dev/$PROJECT_ID/$REPO/$IMAGE:latest || true
                    kubectl apply -f k8s/
                    '''
                }
            }
        }
    }
}
