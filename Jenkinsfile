pipeline {
    agent any

    environment {
        AWS_REGION   = 'ap-south-1'
        CLUSTER_NAME = 'eksdemo1'
        DOCKER_IMAGE = 'bhupal716/flask-todo:latest'
    }

    stages {

        stage('Checkout Code from GitHub') {
            steps {
                git branch: 'main', 
                    credentialsId: 'github-creds', 
                    url: 'https://github.com/bhupal2029/CD-Pipeline.git'
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                    echo "Logging in to Docker Hub..."
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker pull $DOCKER_IMAGE
                    docker logout
                    '''
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
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
                '''
            }
        }

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
            echo "❌ Deployment failed. Check logs for details."
        }
    }
}

