pipeline {
    agent any

    environment {
        AWS_REGION = "ap-south-1"                  
        ECR_REPO = "sample-proj1"               
        AWS_ACCOUNT_ID = "657399551937"           
        IMAGE = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}"
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
                 withAWS(credentials: 'aws-creds', region: "$AWS_REGION") {
                    sh '''
                        echo "AWS credentials configured by withAWS"
                    '''
                
                // Write kubeconfig from Jenkins secret
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG_FILE')]) {
                    sh '''
                        mkdir -p ~/.kube
                        cp $KUBECONFIG_FILE ~/.kube/config
                    '''
                }
            }
        }
     }
        stage('Login to ECR') {
            steps {
                withAWS(credentials: 'aws-creds', region: "$AWS_REGION") {
                    sh '''
                        aws ecr get-login-password --region $AWS_REGION \
                        | docker login --username AWS --password-stdin $IMAGE
                    '''
            }
        }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t $IMAGE:latest .
                '''
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh '''
                    docker push $IMAGE:latest
                '''
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh '''
                    sed -i "s|image:.*|image: $IMAGE:latest|" k8s/deployment.yaml

                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml

                    kubectl rollout status deployment/octoopsacademy-deployment
                '''
            }
        }
    }
}
