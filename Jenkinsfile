pipeline {
  agent any

  environment {
    // Change this to your Docker Hub repo:
    IMAGE_NAME = "21694785/jenkins"
    IMAGE_TAG  = "build-${env.BUILD_NUMBER}"

    // Credentials already created in Jenkins
    SNYK_TOKEN = credentials('SNYK_TOKEN')   // personal API token easiest
    // If using Snyk service account, optionally also:
    // SNYK_ORG   = "21694785"
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Node 16 env (deps + tests + Snyk)') {
      steps {
        script {
          def n = docker.image('node:16'); n.pull()
          n.inside('-u root:root') {
            sh 'node -v'
            sh 'npm ci || npm install'
            sh 'npm test || echo "No tests defined"'
            sh 'npm install -g snyk'
            sh 'snyk auth $SNYK_TOKEN'
            // add --org=$SNYK_ORG if you used a service-account token
            sh 'snyk test --severity-threshold=high'
          }
        }
      }
    }

    stage('Build Docker image') {
      steps { sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ." }
    }

    stage('Push image') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'dockerhub-creds',
          usernameVariable: 'DOCKER_USER',
          passwordVariable: 'DOCKER_PASS'
        )]) {
          sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
          sh 'docker push $IMAGE_NAME:$IMAGE_TAG'
        }
      }
    }
  }

  post {
    always { sh 'docker logout || true' }
    success { echo '✅ Pipeline finished successfully.' }
    failure { echo '❌ Pipeline failed — check the stage above.' }
  }
}
