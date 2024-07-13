pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = 'us-east-2'
        ECR_REPO = '041738715000.dkr.ecr.us-east-2.amazonaws.com/simple-html'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        KUBE_MANIFESTS_REPO = 'https://github.com/Yudiz-Sarjan/simple-html.git'
        AWS_CREDENTIALS = 'aws-creds-id'  // Update with your actual credentials ID
        AWS_ACCOUNT_ID = '041738715000'   // Update with your actual AWS account ID
        KUBECONFIG = "/var/lib/jenkins/.kube/config" // Adjust this path as necessary
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
                script {
                    // Clone the Kubernetes manifests repository
                    git url: "${KUBE_MANIFESTS_REPO}", branch: 'master'

                    // Set KUBECONFIG environment variable
                    withEnv(["KUBECONFIG=${KUBECONFIG}"]) {
                        // Authenticate Docker client to ECR using AWS CLI
                        withCredentials([[$class: 'AmazonWebServicesCredentialsBinding',
                                          accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                                          credentialsId: "${AWS_CREDENTIALS}",
                                          secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
                        
                            // Replace the placeholder ${IMAGE_TAG} in Deployment.yaml with the actual image tag
                            sh "sed -i 's|\\\$IMAGE_TAG|${IMAGE_TAG}|' Deployment.yaml"
                        
                            // Apply Deployment.yaml, Service.yaml, and Ingress.yaml to the EKS cluster
                            sh "kubectl apply -f Deployment.yaml"
                            sh "kubectl apply -f Service.yaml"
                            sh "kubectl apply -f Ingress.yaml"
                        }
                    }
                }
            }
        }
    }
}
