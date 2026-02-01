pipeline {
    agent { label 'docker-host'}

    stages {

        stage('Checkout Repository') {
            steps {
                checkout scm
            }
        }

        stage('Deploy DEV') {
            when {
                branch 'dev'
            }
            steps {
                /* groovylint-disable-next-line GStringExpressionWithinString */
                sh '''
                  echo "DEV branch detected"
                  echo "Deploying Kubernetes manifests from k8s/dev"
                  kubectl apply -f ${WORKSPACE}/k8s/dev/
                '''
            }
        }

        stage('Deploy TEST') {
            when {
                branch 'test'
            }
            steps {
                /* groovylint-disable-next-line GStringExpressionWithinString */
                sh '''
                  echo "TEST branch detected"
                  echo "Deploying Kubernetes manifests from k8s/test"
                  kubectl apply -f ${WORKSPACE}/k8s/test/
                '''
            }
        }

        stage('Deploy PROD') {
            when {
                branch 'prod'
            }
            steps {
                /* groovylint-disable-next-line GStringExpressionWithinString */
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
