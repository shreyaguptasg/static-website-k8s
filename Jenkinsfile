pipeline {
  agent any
  options { timestamps() }

  environment {
    // Use the SAME image everywhere
    DOCKER_IMAGE = "shreyasg123/static-web:latest"
    DOCKER_HUB_CRED = "dockerhub-credentials"

    // Prefer Jenkins user's default kubeconfig if you moved it there earlier:
    // KUBECONFIG = "${env.HOME}/.kube/config"
    // Otherwise use your file path:
    KUBECONFIG = "/home/ec2-user/.kube/config"
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
        sh '''
          docker version
          echo "[Build] Building image: $DOCKER_IMAGE"
          docker build -t $DOCKER_IMAGE .
        '''
      }
    }

    stage('Docker Login & Push') {
      steps {
        withCredentials([usernamePassword(credentialsId: "${DOCKER_HUB_CRED}",
                                          usernameVariable: 'USERNAME',
                                          passwordVariable: 'PASSWORD')]) {
          sh '''
            echo "$PASSWORD" | docker login -u "$USERNAME" --password-stdin
            docker push $DOCKER_IMAGE
            docker logout
          '''
        }
      }
    }

    stage('Kubernetes Deploy') {
      steps {
        sh '''
          export KUBECONFIG="$KUBECONFIG"
          kubectl version --client
          kubectl config current-context || true

          # Render the deployment template with the actual image
          sed "s|{{IMAGE}}|$DOCKER_IMAGE|g" k8s/deployment.yaml.tpl | kubectl apply -f -
          kubectl apply -f k8s/service.yaml

          kubectl rollout status deploy/static-web --timeout=90s
        '''
      }
    }

    stage('Verify') {
      steps {
        sh '''
          export KUBECONFIG="$KUBECONFIG"
          kubectl get deploy static-web -o wide
          kubectl get pods -l app=static-web -o wide
          kubectl get svc static-web-service -o wide
        '''
      }
    }
  }

  post {
    success { echo '✅ CI/CD complete.' }
    failure { echo '❌ Pipeline failed—check logs.' }
  }
}
