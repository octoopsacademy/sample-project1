pipeline {
  agent any
  environment {
    DOCKERHUB = credentials('dockerhub-creds')
    AWS = credentials('aws-creds')
    IMAGE = "<dockerhub-user>/octoopsacademy"
    TAG = "${env.BUILD_NUMBER}-${env.GIT_COMMIT?.substring(0,7)}"
  }
  stages {
    stage('Checkout') { steps { checkout scm } }
    stage('Build') { steps { sh 'docker build -t $IMAGE:$TAG .' } }
    stage('Push') {
      steps {
        sh 'echo $DOCKERHUB_PSW | docker login -u $DOCKERHUB_USR --password-stdin'
        sh 'docker push $IMAGE:$TAG'
        sh 'docker tag $IMAGE:$TAG $IMAGE:latest && docker push $IMAGE:latest'
      }
    }
    stage('Deploy to EKS') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'aws-creds', passwordVariable: 'AWS_SECRET', usernameVariable: 'AWS_KEY')]){
          sh 'aws configure set aws_access_key_id $AWS_KEY && aws configure set aws_secret_access_key $AWS_SECRET && aws configure set default.region us-east-1'
          sh 'aws eks update-kubeconfig --name <your-cluster-name> --region us-east-1'
          sh "kubectl set image deployment/octoopsacademy octoopsacademy=${IMAGE}:${TAG} --record || kubectl apply -f k8s/deployment.yaml"
          sh 'kubectl apply -f k8s/service.yaml'
        }
      }
    }
  }
}
