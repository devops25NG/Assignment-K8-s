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
                sh '''
                  echo "TEST branch detected"
                  echo "Deploying Kubernetes manifests from k8s/test"
                  kubectl apply -f ${WORKSPACE}/k8s/test/
                '''
            }
        }

        stage('Deploy PROD') {
            when {
                branch 'main'
            }
            steps {
                sh '''
                  echo "PROD branch detected"
                  echo "Deploying Kubernetes manifests from k8s/prod"
                  kubectl apply -f ${WORKSPACE}/k8s/prod/
                '''
            }
        }
    }

    post {
        success {
            echo "Deployment completed for branch: ${BRANCH_NAME}"
        }
        failure {
            echo "Deployment failed for branch: ${BRANCH_NAME}"
        }
    }
}
