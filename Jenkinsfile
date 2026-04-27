pipeline {

    parameters {
        string(name: 'IMAGE_TAG', defaultValue: 'latest')
        choice(name: 'ENV', choices: ['dev','stage','prod'])
    }

    agent any

    environment {
        PROJECT_ID = "internal-sandbox-446612"
        REGION = "asia-south1"
        REPO = "my-repo"
        IMAGE = "devops-app"
        CLUSTER = "my-cluster"
        ZONE = "asia-south1-a"
    }

    stages {

        stage('Clone Code') {
            steps {
                git 'https://github.com/Pvramana1/devops-app.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t $REGION-docker.pkg.dev/$PROJECT_ID/$REPO/$IMAGE:$IMAGE_TAG .
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh 'echo "Running tests..."'
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([file(credentialsId: 'gcp-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    sh '''
                    gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS
                    gcloud config set project $PROJECT_ID
                    gcloud auth configure-docker $REGION-docker.pkg.dev -q

                    docker push $REGION-docker.pkg.dev/$PROJECT_ID/$REPO/$IMAGE:$IMAGE_TAG
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

                    kubectl apply -f k8s/
                    '''
                }
            }
        }
    }
}
