pipeline {
    agent any

    environment {
        AWS_REGION   = 'ap-south-1'
        CLUSTER_NAME = 'eksdemo1'                          // <-- Replace with your EKS cluster name if different
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
        stage('Login to Docker Hub') {
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

        /* ---------- DEPLOY TO EKS ---------- */
        stage('Deploy to EKS') {
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

                    echo "Waiting for rollout to complete..."
                    kubectl rollout status deployment/flask-todo-deployment
                    '''
                }
            }
        }

        /* ---------- VERIFY DEPLOYMENT ---------- */
        stage('Verify Deployment') {
            steps {
                sh '''
                echo "============================"
                echo "Verifying Deployment Status"
                echo "============================"
                kubectl get pods -o wide
                kubectl get svc
                kubectl get ingress
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Deployment completed successfully!"
            sh 'kubectl get ingress'
        }
        failure {
            echo "❌ Deployment failed. Please check the Jenkins console logs."
        }
    }
}

