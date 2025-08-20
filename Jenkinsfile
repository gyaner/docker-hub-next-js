pipeline {
  agent any

  environment {
    REPO_URL     = 'https://github.com/gyaner/next-js-hoisting'
    DOCKER_IMAGE = 'gyandevloper/nextjs-app'   // 👈 Your Docker Hub repo
    EC2_USER     = 'ubuntu'
    EC2_HOST     = '43.204.25.229'
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
        branch 'main'    // ✅ Run only if branch == main
      }
      steps {
        script {
          echo "Building Docker image from Docker Hub base..."
          sh """
            # Pull base image from Docker Hub
            docker pull node:18-alpine

            # Build new image for this commit
            SHORT_COMMIT=\$(echo "$GIT_COMMIT" | cut -c1-7)
            IMAGE_TAG="$DOCKER_IMAGE:\$SHORT_COMMIT"

            docker build -t "\$IMAGE_TAG" .
            docker tag "\$IMAGE_TAG" "$DOCKER_IMAGE:latest"

            echo "Pushing image to Docker Hub..."
            docker login -u "$DOCKER_HUB_USER" -p "$DOCKER_HUB_PASS"
            docker push "\$IMAGE_TAG"
            docker push "$DOCKER_IMAGE:latest"
          """
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
