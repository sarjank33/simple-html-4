pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = 'us-east-2'
        ECR_REPO = 'simple-html'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        KUBE_MANIFESTS_REPO = 'https://github.com/Yudiz-Sarjan/simple-html.git'
        AWS_CREDENTIALS = 'aws-creds-id'  // Update with your actual credentials ID
        AWS_ACCOUNT_ID = '041738715000'   // Update with your actual AWS account ID
    }

    stages {
        stage('Build and Push Docker Image') {
            steps {
                script {
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding',
                                      accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                                      credentialsId: "${AWS_CREDENTIALS}",
                                      secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
                        
                        // Log in to AWS ECR
                        sh "aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"

                        // Build and push Docker image
                        def customImage = docker.build("${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_TAG}")
                        customImage.push()
                    }
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                git url: "${KUBE_MANIFESTS_REPO}", branch: 'master', credentialsId: "${GIT_CREDENTIALS}"
                sh "kubectl apply -f Deployment.yaml -f Service.yaml -f Ingress.yaml"
            }
        }
    }
}
