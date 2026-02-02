pipeline {
  agent any

  environment {
    // DockerHub
    DOCKER_USER = "usernamenarendra"
    DOCKER_CRED = "dockerhub-creds"

    // Image names
    FRONTEND_IMAGE = "kubecoin-frontend"
    BACKEND_IMAGE  = "kubecoin-backend"

    // Environment mapping
    ENV_NAME  = "${env.BRANCH_NAME}"
    IMAGE_TAG = "${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
    // Webhook token stored as a Jenkins 'Secret Text' credential. Replace 'webhook-token-id' with your credential id.
    WEBHOOK_TOKEN = credentials('webhook-token-id')
  }

  triggers {
    // Trigger on GitHub push webhooks (requires 'GitHub plugin')
    githubPush()

    // Generic webhook trigger (requires 'Generic Webhook Trigger' plugin).
    // To use: create a secret-text credential 'webhook-token-id' and configure your repo webhook to POST:
    // JENKINS_URL/generic-webhook-trigger/invoke?token=<token>
    GenericTrigger(
      genericVariables: [
        [key: 'ref', value: '$.ref']
      ],
      causeString: 'Triggered on $ref',
      token: "${WEBHOOK_TOKEN}",
      printContributedVariables: true,
      printPostContent: true
    )
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Docker Login') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: DOCKER_CRED,
          usernameVariable: 'DOCKER_USERNAME',
          passwordVariable: 'DOCKER_PASSWORD'
        )]) {
          sh '''
            echo "$DOCKER_PASSWORD" | docker login \
              -u "$DOCKER_USERNAME" --password-stdin
          '''
        }
      }
    }

    stage('Build Docker Images') {
      steps {
        sh """
          docker build -t $DOCKER_USER/$FRONTEND_IMAGE:$IMAGE_TAG frontend/
          docker build -t $DOCKER_USER/$BACKEND_IMAGE:$IMAGE_TAG backend/
        """
      }
    }

    stage('Push Docker Images') {
      steps {
        sh """
          docker push $DOCKER_USER/$FRONTEND_IMAGE:$IMAGE_TAG
          docker push $DOCKER_USER/$BACKEND_IMAGE:$IMAGE_TAG
        """
      }
    }

    stage('Approve Production') {
      when {
        branch 'main'
      }
      steps {
        input message: "Approve deployment of ${IMAGE_TAG} to PRODUCTION?"
      }
    }

    stage('Deploy to Kubernetes') {
      steps {
        sh """
          kubectl set image deployment/kubecoin-frontend \
            kubecoin-frontend=$DOCKER_USER/$FRONTEND_IMAGE:$IMAGE_TAG \
            -n $ENV_NAME

          kubectl set image deployment/kubecoin-backend \
            kubecoin-backend=$DOCKER_USER/$BACKEND_IMAGE:$IMAGE_TAG \
            -n $ENV_NAME
        """
      }
    }
  }

  post {
    always {
      sh 'docker logout || true'
    }
  }
}
