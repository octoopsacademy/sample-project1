pipeline {
    agent any

    environment {
        AWS_REGION = "ap-south-1"                  
        ECR_REPO = "657399551937.dkr.ecr.ap-south-1.amazonaws.com/sample-proj1"               
        AWS_ACCOUNT_ID = "657399551937"           
        IMAGE_TAG = "latest"
    }

    stages {

        stage('Checkout') {
            steps {
                git(
                    branch: 'main',
                    credentialsId: 'github-creds',
                    url: 'https://github.com/octoopsacademy/sample-project1.git'
                )
            }
        }

        stage('Configure AWS & Kubeconfig') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'aws-creds-userpass',
                                                 usernameVariable: 'AWS_ACCESS_KEY_ID',
                                                 passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sh '''
                        # Configure AWS CLI
                        aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
                        aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
                        aws configure set default.region $AWS_REGION
                    '''
                }

                // Copy Kubeconfig file from Jenkins secret
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG_FILE')]) {
                    sh '''
                        mkdir -p ~/.kube
                        cp $KUBECONFIG_FILE ~/.kube/config
                    '''
                }
            }
        }
        stage('Login to ECR') {
            steps {
                sh '''
                    aws ecr get-login-password --region $AWS_REGION \
                    | docker login --username AWS --password-stdin $ECR_REPO
                '''
            }
        }
        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t $ECR_REPO:$IMAGE_TAG -f Dockerfile1 .
                '''
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh '''
                    docker push $ECR_REPO:$IMAGE_TAG
                '''
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh '''
                    sed -i "s|image:.*|image: $ECR_REPO:latest|" k8s/deployment.yaml

                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml

                    kubectl rollout status deployment/octoopsacademy
                '''
            }
        }
    }
}
