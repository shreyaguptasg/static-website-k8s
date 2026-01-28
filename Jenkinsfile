pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "shreyasg123/static-web:latest"   // <— your Docker Hub image
        DOCKER_HUB_CRED = "dockerhub-credentials"       // <— Jenkins me add kiya hua ID
        KUBECONFIG = "/var/lib/jenkins.yaml"          // <— we copied this earlier
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/shreyaguptasg/static-website-k8s.git'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker version'
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
                sh 'export KUBECONFIG=$KUBECONFIG && kubectl apply -f k8s/deployment.yaml'
                sh 'export KUBECONFIG=$KUBECONFIG && kubectl apply -f k8s/service.yaml'
            }
        }

        stage('Verify') {
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
        success { echo '✅ CI/CD complete.' }
        failure { echo '❌ Pipeline failed—check logs.' }
    }
}
