pipeline {
  agent { label 'linux' }

  options {
    skipDefaultCheckout(true)
    // ansiColor('xterm')   // <-- enable only if Pipeline: ANSI Color plugin is installed
    timestamps()
  }

  parameters {
    string(name: 'IMAGE_TAG', defaultValue: "${env.BUILD_NUMBER}", description: 'Tag for Docker image (override if needed)')
    booleanParam(name: 'PUSH_LATEST', defaultValue: true, description: 'Also tag & push: latest (convenience)')
  }

  environment {
    IMAGE_NAME = 'sudotechadmin/chatsocket-app'
    DOCKER_CREDENTIALS_ID = 'dockerhub'   // ensure this exactly matches your Jenkins credential ID
    CONTAINER_NAME = 'chatsocket-app'
    EXPOSE_PORT = '3000'
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
        script {
          def tag = params.IMAGE_TAG
          sh "docker build --pull -t ${env.IMAGE_NAME}:${tag} ."
          if (params.PUSH_LATEST.toBoolean()) {
            sh "docker tag ${env.IMAGE_NAME}:${tag} ${env.IMAGE_NAME}:latest"
          }
        }
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
        script {
          def tag = params.IMAGE_TAG
          sh "docker push ${env.IMAGE_NAME}:${tag}"
          if (params.PUSH_LATEST.toBoolean()) {
            sh "docker push ${env.IMAGE_NAME}:latest"
          }
        }
      }
    }

    stage('Deploy Container') {
      steps {
        script {
          def tag = params.IMAGE_TAG
          def imgWithTag = "${env.IMAGE_NAME}:${tag}"

          // Pull the specific image tag from registry
          sh "docker pull ${imgWithTag} || true"

          // Stop & remove any existing container (use shell single-quoted to avoid Groovy/GString interpolation issues)
          sh '''
            if [ -n "$(docker ps -q -f name=$CONTAINER_NAME)" ]; then
              echo "Stopping running container..."
              docker stop $CONTAINER_NAME || true
            fi
            if [ -n "$(docker ps -aq -f name=$CONTAINER_NAME)" ]; then
              echo "Removing previous container..."
              docker rm $CONTAINER_NAME || true
            fi
          '''

          // Run new container
          sh "docker run -d --name ${env.CONTAINER_NAME} -p ${env.EXPOSE_PORT}:${env.EXPOSE_PORT} ${imgWithTag}"
        }
      }
    }

    stage('Cleanup') {
      steps {
        sh "docker image prune -af || true"
      }
    }
  }

  post {
    success {
      echo "SUCCESS: deployed ${env.IMAGE_NAME}:${params.IMAGE_TAG} as ${env.CONTAINER_NAME}"
    }
    failure {
      echo "FAILED: see console for details"
    }
  }
}
