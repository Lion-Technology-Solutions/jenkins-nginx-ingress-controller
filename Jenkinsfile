pipeline {
    agent any
    
    environment {
        CLUSTER_NAME = "prod"
        AWS_REGION = "us-east-2"
        HELM_CHART_VERSION = "4.10.1"
    }
    
    stages {
        stage('Configure AWS & EKS') {
            steps {
        sh """
        
        aws sts get-caller-identity
        aws eks update-kubeconfig --region ${AWS_REGION} --name ${CLUSTER_NAME}
        """
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
    
    post {
        always {
            echo 'Pipeline completed - cleaning up workspace'
            cleanWs()
        }
        success {
            echo 'NGINX Ingress Controller deployed successfully!'
        }
        failure {
            echo 'Pipeline failed - check logs for details'
        }
    }
}