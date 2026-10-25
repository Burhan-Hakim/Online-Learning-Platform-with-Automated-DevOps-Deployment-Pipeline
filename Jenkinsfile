pipeline {
  agent any
  environment {
    IMAGE = 'hakimburhan/learning-platform'
    HOSTS = '44.193.76.73 3.93.65.137'
  }
  stages {
    stage('Checkout') {
      steps {
        checkout scm
        sh 'git log -1 --oneline'
      }
    }
    stage('Validate') {
      steps { sh 'test -f Dockerfile && ls -la' }
    }
    stage('Build Image') {
      steps { sh 'docker build -t $IMAGE:$BUILD_NUMBER -t $IMAGE:latest .' }
    }
    stage('Push Image') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DH_USER', passwordVariable: 'DH_PASS')]) {
          sh 'echo "$DH_PASS" | docker login -u "$DH_USER" --password-stdin'
          sh 'docker push $IMAGE:$BUILD_NUMBER && docker push $IMAGE:latest'
        }
      }
    }
    stage('Deploy') {
      steps {
        sh '''
          for h in $HOSTS; do
            ssh -o StrictHostKeyChecking=no ubuntu@$h "
              docker pull $IMAGE:latest &&
              (docker rm -f learning-app || true) &&
              docker run -d --name learning-app --restart always --env-file /opt/learning-app/db.env -p 8080:80 $IMAGE:latest"
          done
        '''
      }
    }
    stage('Verify') {
      steps {
        sh '''
          sleep 10
          for h in $HOSTS; do
            echo "== $h =="
            ssh ubuntu@$h "docker ps --filter name=learning-app --format '{{.Names}} {{.Image}} {{.Status}}' && curl -s -o /dev/null -w 'HTTP %{http_code}\\n' http://localhost"
          done
        '''
      }
    }
  }
}
