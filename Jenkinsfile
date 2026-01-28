pipeline {
    agent any

    environment {
        // Docker image name (Docker Hub)
        DOCKER_IMAGE = "shreyasg123/static-web:latest"

        // Jenkins credential ID you created for Docker Hub
        DOCKER_HUB_CRED = "dockerhub-credentials"

        // KUBECONFIG path (we copied k3s.yaml to ec2-user home earlier)
        KUBECONFIG = "/home/ec2-user/k3s.yaml"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/<your-github-username>/static-website-k8s.git'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker version || (echo "Docker not found" && exit 1)'
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('Docker Login & Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: "$DOCKER_HUB_CRED",
                        usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh '''
                        echo "$PASSWORD" | docker login -u "$USERNAME" --password-stdin
                        docker push $DOCKER_IMAGE
                    '''
                }
            }
        }

        stage('Kubernetes Deploy') {
            steps {
                // Make sure kubeconfig is set for this shell
                sh 'export KUBECONFIG=$KUBECONFIG && kubectl version --short'
                sh 'export KUBECONFIG=$KUBECONFIG && kubectl apply -f k8s/deployment.yaml'
                sh 'export KUBECONFIG=$KUBECONFIG && kubectl apply -f k8s/service.yaml'
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    export KUBECONFIG=$KUBECONFIG
                    kubectl get deploy static-web
                    kubectl get pods -l app=static-web
                    kubectl get svc static-web-service
                '''
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed—check the stage logs.'
        }
    }
}
