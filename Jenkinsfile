pipeline {
  agent any

  environment {
    REPO_URL     = 'https://github.com/gyaner/next-js-hoisting'
    DOCKER_IMAGE = 'gyandevloper/nextjs-app'
    EC2_USER     = 'ubuntu'
    EC2_HOST     = '65.2.166.236'
    APP_DIR      = '/opt/nextjs-app'
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build & Push Docker Image') {
      when {
        branch 'main'
      }
      steps {
        script {
          withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_HUB_USER', passwordVariable: 'DOCKER_HUB_PASS')]) {
            sh """
              docker pull node:18-alpine

              SHORT_COMMIT=\$(echo "$GIT_COMMIT" | cut -c1-7)
              IMAGE_TAG="$DOCKER_IMAGE:\$SHORT_COMMIT"

              docker build -t "\$IMAGE_TAG" .
              docker tag "\$IMAGE_TAG" "$DOCKER_IMAGE:latest"

              echo "Logging in to Docker Hub..."
              echo "\$DOCKER_HUB_PASS" | docker login -u "\$DOCKER_HUB_USER" --password-stdin

              docker push "\$IMAGE_TAG"
              docker push "$DOCKER_IMAGE:latest"
            """
          }
        }
      }
    }

    stage('Deploy to EC2') {
      when {
        branch 'main'
      }
      steps {
        sshagent(['ec2-ssh-key']) {
          sh """
            ssh -o StrictHostKeyChecking=no $EC2_USER@$EC2_HOST '
              set -e
              docker pull $DOCKER_IMAGE:latest
              docker stop nextjs-main || true
              docker rm nextjs-main || true
              docker run -d -p 3000:3000 --name nextjs-main $DOCKER_IMAGE:latest
            '
          """
        }
      }
    }
  }
}
