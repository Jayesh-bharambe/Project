pipeline {
    agent any

    environment {
        AWS_REGION = "ap-south-1"
        ECR_URL = credentials('ecr-url')   // ECR repo URL
        IMAGE = "${ECR_URL}:latest"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh """
                pip install -r requirements.txt
                """
            }
        }

        stage('Run Tests') {
            steps {
                echo "No tests included, skipping..."
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                docker build -t prediction-api .
                docker tag prediction-api:latest ${IMAGE}
                """
            }
        }

        stage('Login to ECR') {
            steps {
                withAWS(credentials: 'aws-creds', region: "${AWS_REGION}") {
                    sh '''
                    aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_URL
                    '''
                }
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh """
                docker push ${IMAGE}
                """
            }
        }

        stage('Deploy to EKS') {
            steps {
                withAWS(credentials: 'aws-creds', region: "${AWS_REGION}") {
                    sh """
                    aws eks update-kubeconfig --name eks-cluster --region ${AWS_REGION}
                    kubectl set image deployment/prediction-api prediction-api=${IMAGE}
                    kubectl rollout restart deployment/prediction-api
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Deployment completed successfully!"
        }
        failure {
            echo "Pipeline failed!"
        }
    }
}
