pipeline {
  agent any

  environment {
    PROJECT_ID = 'omega-byte-460612-p8'
    IMAGE = "gcr.io/${PROJECT_ID}/demo-app"
    CREDENTIALS_ID = 'jenkins-gcp-sa-key'
    GKE_CLUSTER = 'jenkins-demo-cluster'
    GKE_ZONE = 'us-central1-a'
  }

  stages {
    stage('Clone Code') {
      steps {
        git branch: 'main', url: 'https://github.com/yaswanth70/jenkins.git'
      }
    }

    stage('Build, Authenticate & Push Docker Image') {
      steps {
        script {
          withCredentials([file(credentialsId: "${CREDENTIALS_ID}", variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
            docker.image('google/cloud-sdk:latest').inside(args: '-v /var/run/docker.sock:/var/run/docker.sock -u root') {
              sh """
                docker build -t ${IMAGE} .
                gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS
                gcloud auth configure-docker --quiet
                docker push ${IMAGE}
              """
            }
          }
        }
      }
    }

    stage('Deploy to GKE') {
      steps {
        script {
          withCredentials([file(credentialsId: "${CREDENTIALS_ID}", variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
            docker.image('google/cloud-sdk:latest').inside(args: '-u root') {
              sh """
                gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS
                gcloud container clusters get-credentials $GKE_CLUSTER --zone $GKE_ZONE --project $PROJECT_ID
                kubectl apply -f deployment.yaml
              """
            }
          }
        }
      }
    }
  }
}
