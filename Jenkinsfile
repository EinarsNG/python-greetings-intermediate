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
          echo "Build $GIT_COMMIT"
          echo "Building python-greetings-app"
          build_docker("einarsngalejs/python-greetings-app:$GIT_COMMIT", "Dockerfile")
        }
      }
    }
    stage('deploy-dev') {
      steps {
        script {
          deploy("dev")
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
          deploy("prod")
        }
      }
    }
    stage('test-prod') {
      steps {
        script {
          test("PRD")
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
  sh "docker push $tag"
}

def test(String env) {
  echo "Testing environment... $env"
  sh "docker run --network=host -t -d --name api_tests_runner_$env einarsngalejs/api-tests-runner:latest"
  sh "docker exec api_tests_runner_$env cucumber PLATFORM=$env --format html --out test-output/report.html
  sh "docker cp api_tests_runner_$env:/api-tests/test-output/report.html report_$env.html"
  sh "docker rm -rf api_tests_runner_$env"
}

def deploy(String env) {
  echo "Deploying environment... $env"
  sh "kubectl set image deployment python-greetings-$env python-greetings-$env-pod=einarsngalejs/python-greetings-app:$GIT_COMMIT"
  //sh "kubectl scale deploy python-greetings-$env --replicas=0"
  //sh "kubectl scale deploy python-greetings-$env --replicas=2"
}
