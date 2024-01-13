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
        script {
          echo "Building python-greetings-app"
          build_docker("einarsngalejs/python-greetings-app", "Dockerfile")
        }
      }
    }
    stage('deploy-dev') {
      steps {
        script {
          deploy("DEV")
        }
      }
    }
    stage('test-dev') {
      steps {
        script {
          test("DEV")
        }
      }
    }
    stage('approval') {
      steps {
        echo "waiting for approval"
      }
    }
    stage('deploy-prod') {
      steps {
        script {
          deploy("PROD")
        }
      }
    }
    stage('test-prod') {
      steps {
        script {
          test("DEV")
        }
      }
    }
  }
  post {
    failure {
      script {
        echo "Pipeline failed... sending notification"
        // invoke discord plugin
      }
    }
    cleanup {
      echo "Cleaning up..."
    }
  }
}

def build_docker(String tag, String file) {
  echo "Building $tag image for api-tests"
  sh "docker build --no-cache -t $tag . -f $file"
  sh "docker login -u $DOCKER_USER -p \"$DOCKER_PASS\""
  sh "docker push $tag:latest"
}

def test(String env) {
  echo "Testing environment... $env"
  // docker run
  // docker exec
  // docker cp
  // extract report logic
  // docker cleanup
}

def deploy(String env) {
  echo "Deploying environment... $env"
  sh "kubectl set image deployment python-greetings-$env python-greetings-$env-pod=einarsngalejs/python-greetings-app:latest"
}
