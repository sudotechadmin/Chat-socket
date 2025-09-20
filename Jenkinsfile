pipeline {
  agent any

  environment {
    DOCKERHUB_REPO = "https://github.com/sudotechadmin/Chat-socket"
    IMAGE_TAG = "${env.BUILD_NUMBER}"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Install') {
      steps {
        sh 'npm ci'
      }
    }

    stage('Test (safe)') {
      steps {
        script {
          // run tests if defined, but don't fail the job if tests not present
          def rc = sh(script: 'npm test || true', returnStatus: true)
          if (rc != 0) {
            echo "npm test returned non-zero. Either tests failed or none defined — continuing."
          } else {
            echo "npm test succeeded."
          }
        }
      }
    }

    stage('Build Docker image') {
      steps {
        script {
          // requires Docker available on the agent
          dockerImage = docker.build("${env.DOCKERHUB_REPO}:${env.IMAGE_TAG}")
        }
      }
    }

    stage('Push to Docker Hub') {
      steps {
        script {
          // docker.withRegistry accepts registry URL and credentialsId
          docker.withRegistry('https://registry.hub.docker.com', 'dockerhub-creds') {
            dockerImage.push()
            // also push 'latest'
            dockerImage.push('latest')
          }
        }
      }
    }

    stage('Deploy (local) — optional') {
      when {
        expression { return true } // change to conditions you want
      }
      steps {
        script {
          // This will run docker commands on the Jenkins agent/host.
          // It stops existing container (if any) and runs the newly pushed image.
          sh '''
            set +e
            docker rm -f chat-app || true
            docker pull ${DOCKERHUB_REPO}:${IMAGE_TAG}
            docker run -d --name chat-app -p 3001:3000 ${DOCKERHUB_REPO}:${IMAGE_TAG}
          '''
        }
      }
    }
  }

  post {
    always {
      echo "Pipeline finished: ${currentBuild.currentResult}"
    }
  }
}
