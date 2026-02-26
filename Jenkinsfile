pipeline {
    agent any
    environment {
        DOCKER_HUB_USER = 'sans97'
        IMAGE_NAME      = 'trend-app'
        IMAGE_TAG       = "${BUILD_NUMBER}"
        EKS_CLUSTER     = 'trend-eks-cluster'
        AWS_REGION      = 'ap-south-1'
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Vennilavan12/Trend.git'
            }
        }
        stage('Docker Build') {
            steps {
                sh """
                    docker build -t ${DOCKER_HUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} .
                    docker tag ${DOCKER_HUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} ${DOCKER_HUB_USER}/${IMAGE_NAME}:latest
                """
            }
        }
        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push ${DOCKER_HUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}
                        docker push ${DOCKER_HUB_USER}/${IMAGE_NAME}:latest
                    """
                }
            }
        }
        stage('Deploy to EKS') {
            steps {
                sh """
                    aws eks update-kubeconfig --name ${EKS_CLUSTER} --region ${AWS_REGION}
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml
                    kubectl rollout status deployment/trend-deployment
                """
            }
        }
    }
    post {
        success { echo 'Deployment Successful!' }
        failure { echo 'Pipeline Failed!' }
    }
}
