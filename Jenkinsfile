pipeline {
    agent { label 'docker-host' }

    environment {
        DOCKER_USER = 'usernamenarendra'
        DOCKER_CRED = 'dockerhub-creds'

        FRONTEND_IMAGE = 'kubecoin-frontend'
        BACKEND_IMAGE  = 'kubecoin-backend'
   
    }

    triggers {
        githubPush()
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Set Environment Namespace') {
            steps {
                script {
                    /* groovylint-disable-next-line NoDef, VariableTypeRequired */
                    def namespaceMap = ['main': 'prod', 'dev': 'dev', 'test': 'test']
                    env.K8S_NAMESPACE = namespaceMap[env.BRANCH_NAME] ?: error("Unsupported branch: ${env.BRANCH_NAME}")
                }
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
          docker build -t $DOCKER_USER/$FRONTEND_IMAGE:${K8S_NAMESPACE} frontend/
          docker build -t $DOCKER_USER/$BACKEND_IMAGE:${K8S_NAMESPACE} backend/
        """
            }
        }

        stage('Push Docker Images') {
            steps {
                sh """
          docker push $DOCKER_USER/$FRONTEND_IMAGE:${K8S_NAMESPACE}
          docker push $DOCKER_USER/$BACKEND_IMAGE:${K8S_NAMESPACE}
        """
            }
        }

        stage('Approve Production') {
            when { branch 'main' }
            steps {
                input message: "Approve deployment of ${K8S_NAMESPACE} to PRODUCTION?"
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh """
        kubectl apply -f k8s/${K8S_NAMESPACE}/

      kubectl set image deployment/frontend \
        frontend=$DOCKER_USER/kubecoin-frontend:${K8S_NAMESPACE} \
        -n ${K8S_NAMESPACE}

      kubectl set image deployment/backend \
        backend=$DOCKER_USER/kubecoin-backend:${K8S_NAMESPACE} \
        -n ${K8S_NAMESPACE}

      kubectl rollout status deployment/frontend -n ${K8S_NAMESPACE}
      kubectl rollout status deployment/backend -n ${K8S_NAMESPACE}
        """
            }
        }
    }
}