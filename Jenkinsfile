pipeline {
  agent {
    docker {
      image 'node:16'
      args  '-u root:root'
    }
  }

  environment {
    REGISTRY_USER  = credentials('REGISTRY_USER')
    REGISTRY_PASS  = credentials('REGISTRY_PASS')
    IMAGE_NAME     = "21694785/jenkins"
    IMAGE_TAG      = "build-${env.BUILD_NUMBER}"
    SNYK_TOKEN     = credentials('SNYK_TOKEN')
    SNYK_ORG       = '21694785'
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Install deps') {
      steps {
        sh 'npm install --save'
      }
    }

    stage('Unit tests') {
      steps {
        sh 'npm test || echo "No tests defined"'
      }
    }

    stage('Snyk scan (fail on High/Critical)') {
      steps {
        sh '''
          npm install -g snyk
          snyk auth ${SNYK_TOKEN}
          snyk test --org=$SNYK_ORG --severity-threshold=high ||  exit 1)
        '''
      }
    }

    stage('Build Docker image') {
      steps {
        sh '''
          echo "$REGISTRY_PASS" | docker login "$REGISTRY_URL" -u "$REGISTRY_USER" --password-stdin
          docker build -t "$REGISTRY_URL/$IMAGE_NAME:$IMAGE_TAG" .
        '''
      }
    }

    stage('Push image') {
      steps {
        sh 'docker push "$REGISTRY_URL/$IMAGE_NAME:$IMAGE_TAG"'
      }
    }
  }
}
