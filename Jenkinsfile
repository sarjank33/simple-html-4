pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = 'us-east-2'
        ECR_REPO = 'simple-html'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        KUBE_MANIFESTS_REPO = 'https://github.com/Yudiz-Sarjan/simple-html.git'
        AWS_CREDENTIALS = 'aws-creds-id'  // Update with your actual credentials ID
        GIT_CREDENTIALS = 'git-creds-id'  // Update with your actual Git credentials ID
    }

    stages {
        stage('Build and Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://041738715000.dkr.ecr.us-east-2.amazonaws.com', 'ecr:us-east-2') {
                        def customImage = docker.build("041738715000.dkr.ecr.us-east-2.amazonaws.com/${ECR_REPO}:${IMAGE_TAG}")
                        customImage.push()
                    }
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                script {
                    git url: "${KUBE_MANIFESTS_REPO}", branch: 'master', credentialsId: "${GIT_CREDENTIALS}"
                    sh "kubectl apply -f Deployment.yaml -f Service.yaml -f Ingress.yaml"
                }
            }
        }
    }
}
