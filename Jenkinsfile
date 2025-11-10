pipeline {
  agent any

  environment {
    IMAGE = "your-dockerhub-username/devops-demo"
    DOCKER_CRED = credentials('dockerhub-creds')   // docker username/password
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/<your-github-username>/devops-demo.git'
      }
    }

    stage('Build') {
      steps {
        sh 'docker --version'
        sh 'docker build -t ${IMAGE}:latest .'
      }
    }

    stage('Test (smoke)') {
      steps {
        sh 'docker run --rm -d --name temp-devops-test -p 3000:3000 ${IMAGE}:latest || true'
        // wait a bit, then test
        sh 'sleep 3'
        sh 'curl -f http://localhost:3000/ || (docker logs temp-devops-test && false)'
        sh 'docker stop temp-devops-test || true'
      }
    }

    stage('Push to Docker Hub') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
          sh 'docker tag ${IMAGE}:latest ${IMAGE}:${BUILD_NUMBER}'
          sh 'docker push ${IMAGE}:latest'
          sh 'docker push ${IMAGE}:${BUILD_NUMBER}'
        }
      }
    }
  }

  post {
    always {
      sh 'docker image prune -f || true'
    }
    success {
      echo "Build succeeded: ${env.BUILD_URL}"
    }
    failure {
      echo "Build FAILED: ${env.BUILD_URL}"
    }
  }
}
