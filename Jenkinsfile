pipeline {
  agent { label 'linux' } 
  options {
    skipDefaultCheckout(true)
    ansiColor('xterm')
    timestamps()
  }
  parameters {
    string(name: 'IMAGE_TAG', defaultValue: "${env.BUILD_NUMBER}", description: 'Tag for Docker image (override if needed)')
  }
  environment {
    IMAGE_NAME = 'sudotechadmin/chatsocket-app' 
    DOCKER_CREDENTIALS_ID = 'dockerhub'
    CONTAINER_NAME = 'chatsocket-app'
 }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
        sh 'ls -la'
      }
    }

    stage('Docker Info') {
      steps {
        sh 'docker --version || true'
        sh 'docker info || true'
      }
    }

    stage('Build Image') {
      steps {
        sh """
          docker build --pull -t ${IMAGE_NAME}:${params.IMAGE_TAG} .
        """
      }
    }

    stage('Docker Login') {
      steps {
        withCredentials([usernamePassword(credentialsId: env.DOCKER_CREDENTIALS_ID,
                                          usernameVariable: 'DOCKER_USER',
                                          passwordVariable: 'DOCKER_PASS')]) {
          sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
        }
      }
    }

    stage('Push Image') {
      steps {
        sh "docker push ${IMAGE_NAME}:${params.IMAGE_TAG}"
      }
    }

    stage('Deploy Container') {
      steps {
        script {       // Stop & remove old container if running
          sh """
              if [ \$(docker ps -q -f name=${CONTAINER_NAME}) ]; then
                  docker stop ${CONTAINER_NAME}
                  docker rm ${CONTAINER_NAME}
              fi
          """
 // Run new container from latest image
          sh """
              docker run -d --name ${CONTAINER_NAME} -p 3000:3000 ${IMAGE_NAME}:latest
          """
                }
            }
        }
  }

  post {
    success {
      echo "SUCCESS: pushed ${IMAGE_NAME}:${params.IMAGE_TAG}"
    }
    failure {
      echo "FAILED: see console for details"
    }
  }
}
