pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = 'us-east-2'
        ECR_REPO = 'simple-html'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        KUBE_MANIFESTS_REPO = 'https://github.com/Yudiz-Sarjan/simple-html.git'
        EKS_CLUSTER_NAME = 'eks-cluster'
    }

    stages {
        stage('Build and Push Docker Image') {
            steps {
                script {
                    docker.build("ecr-url/${ECR_REPO}:${IMAGE_TAG}")
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'aws-creds-id', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
                        docker.withRegistry('', 'ecr:us-east-2') {
                            docker.image("ecr-url/${ECR_REPO}:${IMAGE_TAG}").push()
                        }
                    }
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                git url: "${KUBE_MANIFESTS_REPO}", branch: 'main'
                sh "kubectl apply -f Deployment.yaml -f Service.yaml -f Ingress.yaml"
            }
        }
    }
}
