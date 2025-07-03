pipeline {
    agent any
    environment {
        CLUSTER_NAME = "prod"
        AWS_REGION = "us-east-2"
        HELM_CHART_VERSION = "4.10.1"
    }
   stage('Configure AWS & EKS') {
    steps {
        withCredentials([[
            $class: 'AmazonWebServicesCredentialsBinding',
            credentialsId: 'AWS_CREDENTIALS',  // Must match credential ID
            accessKeyVariable: 'AWS_ACCESS_KEY_ID',
            secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
        ]]) {
            sh """
            aws configure set aws_access_key_id ${AWS_ACCESS_KEY_ID}
            aws configure set aws_secret_access_key ${AWS_SECRET_ACCESS_KEY}
            aws eks update-kubeconfig --region ${AWS_REGION} --name ${CLUSTER_NAME}
            """
        }
    }
}
        stage('Add Helm Repo') {
            steps {
                sh """
                helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
                helm repo update
                """
            }
        }
        stage('Deploy Ingress Controller') {
            steps {
                sh """
                helm upgrade --install ingress-nginx ingress-nginx/ingress-nginx \
                    --version ${HELM_CHART_VERSION} \
                    --namespace ingress-nginx \
                    --create-namespace \
                    --set controller.service.type=LoadBalancer \
                    --set "controller.service.annotations.service\\.beta\\.kubernetes\\.io/aws-load-balancer-type=nlb" \
                    --set "controller.service.annotations.service\\.beta\\.kubernetes\\.io/aws-load-balancer-scheme=internet-facing"
                """
            }
        }
        stage('Verify Deployment') {
            steps {
                sh """
                kubectl -n ingress-nginx get pods
                kubectl -n ingress-nginx get svc
                """
            }
        }
    }
}