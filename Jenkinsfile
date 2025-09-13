pipeline {
    agent none

    environment {
        AWS_REGION = "us-west-2"
        ACCOUNT_ID = "784074784226"
        ECR_REPO = "service-a"   // change for each service
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            agent { label 'node1' }
            steps {
                git branch: 'main', url: 'https://github.com/VinayAlt/service-a.git'
            }
        }

        stage('Build Docker Image') {
            agent { label 'node1' }
            steps {
                sh 'docker build -t $ECR_REPO:$IMAGE_TAG .'
            }
        }

        stage('Tag Image') {
            agent { label 'node1' }
            steps {
                sh 'docker tag $ECR_REPO:$IMAGE_TAG $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPO:$IMAGE_TAG'
            }
        }

        stage('Push to ECR') {
            agent { label 'node1' }
            steps {
                withAWS(credentials: 'aws-creds', region: "${AWS_REGION}") {
                    sh 'aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com'
                    sh 'docker push $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPO:$IMAGE_TAG'
                }
            }
        }

        stage('Deploy to Kubernetes') {
            agent { label 'built-in' }
            steps {
                sh '''
                # Update deployment file with new image tag
                sed -i "s/service-a:latest/service-a:${IMAGE_TAG}/g" k8s/deploy.yaml

                # Apply deployment and service
                kubectl apply -f k8s/deploy.yaml -n default
                kubectl apply -f k8s/service.yaml -n default
                kubectl apply -f k8s/ingress.yaml -n default

                # Wait until rollout is complete
                kubectl rollout status deployment/service-a -n default
            '''
            }
        }
    }
}

