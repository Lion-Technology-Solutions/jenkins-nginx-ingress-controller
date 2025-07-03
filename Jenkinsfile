pipeline {
    agent any
    environment {
        CLUSTER_NAME = 'prod'
        AWS_REGION = 'us-east-2'
        HELM_CHART_VERSION = '4.10.1'  //# Check latest version: https://artifacthub.io/packages/helm/ingress-nginx/ingress-nginx
    }
    stages {
        stage('Configure AWS CLI') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'AWS_CREDENTIALS',
                    accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                    secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                ]]) {
                    sh """
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
                helm upgrade --install ingress-nginx ingress-nginx/ingress-nginx \\
                    --version ${HELM_CHART_VERSION} \\
                    --namespace ingress-nginx \\
                    --create-namespace \\
                    --set controller.service.type=LoadBalancer \\
                    --set controller.service.annotations."service\.beta\.kubernetes\.io/aws-load-balancer-type"="nlb" \\
                    --set controller.service.annotations."service\.beta\.kubernetes\.io/aws-load-balancer-scheme"="internet-facing" \\
                    --set controller.service.annotations."service\.beta\.kubernetes\.io/aws-load-balancer-ssl-cert"="${AWS_CERT_ARN}" \\
                    --set controller.service.annotations."service\.beta\.kubernetes\.io/aws-load-balancer-backend-protocol"="http" \\
                    --set controller.service.annotations."service\.beta\.kubernetes\.io/aws-load-balancer-ssl-ports"="https" \\
                    --set controller.service.targetPorts.https=http \\
                    --set controller.service.ports.https=443
                """
            }
        }
        
        stage('Verify Deployment') {
            steps {
                sh """
                kubectl -n ingress-nginx get pods -w
                kubectl -n ingress-nginx get svc
                """
            }
        }
    }
}