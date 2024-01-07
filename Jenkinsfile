pipeline {
  agent any
  triggers{
    pollSCM("* * * * *")
  }
  environment{
    DOCKER_USER = credentials("docker-user")
    DOCKER_PASS = credentials("docker-pw")
  }
  stages {
    stage('build-app') {
      steps {
        echo "building app"
      }
    }
    stage('deploy-dev') {
      steps {
        echo "deploying app"
      }
    }
    stage('test-dev') {
      steps {
        echo "testing app"
      }
    }
    stage('approval') {
      steps {
        echo "waiting for approval"
      }
    }
    stage('deploy-prod') {
      steps {
        echo "deploying app prod"
      }
    }
    stage('test-prod') {
      steps {
        echo "testing app prod"
      }
    }
  }
}
