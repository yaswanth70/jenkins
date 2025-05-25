pipeline {
  agent any

  environment {
    PROJECT_ID = 'omega-byte-460612-p8'
    IMAGE = "us-central1-docker.pkg.dev/${PROJECT_ID}/my-repo/demo-app"
    CREDENTIALS_ID = 'jenkins-gcp-sa-key'
  }

  stages {
    stage('Clone Code') {
      steps {
        git branch: 'main', url: 'https://github.com/yaswanth70/jenkins.git'
      }
    }

    stage('Build Image') {
      steps {
        script {
          docker.build("${IMAGE}")
        }
      }
    }

    stage('Push Image to GCR') {
      steps {
        withCredentials([file(credentialsId: "${CREDENTIALS_ID}", variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
          sh '''
            gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS
            gcloud auth configure-docker us-central1-docker.pkg.dev
            gcloud auth print-access-token | docker login -u oauth2accesstoken --password-stdin https://us-central1-docker.pkg.dev
            docker push ${IMAGE}
          '''
        }
      }
    }

    stage('Deploy to GKE') {
      agent {
        docker {
          image 'gcr.io/google.com/cloudsdktool/cloud-sdk:latest'
          args '-u root:root'
        }
      }
      steps {
        withCredentials([file(credentialsId: "${CREDENTIALS_ID}", variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
          sh '''
            gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS
            gcloud container clusters get-credentials jenkins-demo-cluster --zone us-central1-a
            kubectl apply -f deployment.yaml
          '''
        }
      }
    }
  }
}
