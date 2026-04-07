pipeline {
    agent any

    environment {
        AWS_REGION      = 'us-east-1'
        AWS_ACCOUNT_ID  = '226363864691'
        ECR_REPO        = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/k8s-demo-app"
        IMAGE_TAG       = "${BUILD_NUMBER}"
        CLUSTER_NAME    = 'my-k8s-cluster'
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
                echo '✅ Code pulled from GitHub'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${ECR_REPO}:${IMAGE_TAG} ."
                echo '✅ Docker image built'
            }
        }

        stage('Push to ECR') {
            steps {
                sh """
                    aws ecr get-login-password --region ${AWS_REGION} | \
                    docker login --username AWS --password-stdin ${ECR_REPO}

                    docker push ${ECR_REPO}:${IMAGE_TAG}
                """
                echo '✅ Image pushed to ECR'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh """
                    aws eks update-kubeconfig --region ${AWS_REGION} --name ${CLUSTER_NAME}

                    sed -i 's|PLACEHOLDER_IMAGE|${ECR_REPO}:${IMAGE_TAG}|g' k8s/deployment.yaml

                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml

                    kubectl rollout status deployment/k8s-demo-app
                """
                echo '✅ Deployed to Kubernetes'
            }
        }
    }

    post {
        success { echo "🎉 Build #${BUILD_NUMBER} deployed successfully!" }
        failure {
            echo "❌ Failed! Rolling back..."
            sh 'kubectl rollout undo deployment/k8s-demo-app || true'
        }
    }
}
