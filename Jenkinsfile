pipeline {
    agent any

    environment {
        AWS_REGION   = 'ap-south-1'
        CLUSTER_NAME = 'eksdemo1'                          // <-- Replace if your EKS cluster name differs
        DOCKER_IMAGE = 'bhupal716/flask-todo:latest'
    }

    stages {

        /* ---------- CHECKOUT FROM GITHUB ---------- */
        stage('Checkout Code from GitHub') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-creds',
                    url: 'https://github.com/bhupal2029/CD-Pipeline.git'
            }
        }

        /* ---------- LOGIN TO DOCKER HUB ---------- */
        stage('Login & Pull Image from Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    echo "============================"
                    echo "Logging into Docker Hub..."
                    echo "============================"
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker pull $DOCKER_IMAGE
                    docker logout
                    '''
                }
            }
        }

        /* ---------- DEPLOY + VERIFY ON EKS ---------- */
        stage('Deploy and Verify on EKS') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-jenkins-creds']]) {
                    sh '''
                    echo "============================"
                    echo "Configuring kubeconfig for EKS"
                    echo "============================"
                    aws eks update-kubeconfig --region $AWS_REGION --name $CLUSTER_NAME

                    echo "============================"
                    echo "Applying Kubernetes Manifests"
                    echo "============================"
                    kubectl apply -f k8s/deployment.yml
                    kubectl apply -f k8s/service.yml
                    kubectl apply -f k8s/ingress.yml

                    echo "============================"
                    echo "Waiting for rollout to complete..."
                    echo "============================"
                    kubectl rollout status deployment/flask-todo-deployment

                    echo "============================"
                    echo "Verifying Deployment Status"
                    echo "============================"
                    kubectl get pods -o wide
                    kubectl get svc
                    echo "============================"
                    echo "Ingress and ALB URL"
                    echo "============================"
                    kubectl get ingress -o wide
                    echo ""
                    echo "ALB External URL:"
                    kubectl get ingress flask-todo-ingress -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
                    echo ""
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ Deployment completed successfully!"
        }
        failure {
            echo "❌ Deployment failed. Please check the Jenkins console logs."
        }
    }
}
