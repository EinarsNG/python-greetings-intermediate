pipeline {
  agent any
  triggers {
    pollSCM("* * * * *")
  }
  environment {
    DOCKER_USER = credentials("docker-user")
    DOCKER_PASS = credentials("docker-pw")
    WEBHOOK_URL = credentials("discord-webhook")
  }
  parameters {
    choice(name: 'PROD_DEPLOY', choices: ['Yes', 'No'], description: 'Deploy to production?')
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
      agent none
      input {
          message "How long to wait?"
          submitter "egalejs"
          parameters {
              choice(name: 'DEPLOY_DELAY', choices: ['0', '1', '5', '10'], description: 'Minutes to wait before deploy to prod?')
          }
      }
      steps {
        script {
          echo "waiting for approval"
          //def deploymentSleepDelay = input id: 'Deploy', message: 'How long to wait?', submitter: 'egalejs',
          //                              parameters: [choice(choices: ['0', '1', '5', '10'], description: 'Minutes to wait before deploy to prod?', name: "DEPLOY_DELAY")]
          sleep time: DEPLOY_DELAY.toInteger(), unit: 'MINUTES'
        }
      }
    }
    stage('deploy-prod') {
      when {
        expression { params.PROD_DEPLOY == 'Yes' }
      }
      steps {
        script {
          deploy("prod")
        }
      }
    }
    stage('test-prod') {
      when {
        expression { params.PROD_DEPLOY == 'Yes' }
      }
      steps {
        script {
          test("PRD")
        }
      }
    }
  }
  post {
    always {
      notify()
    }
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
  //sh "docker run --network=host -t -d --name api_tests_runner_$env einarsngalejs/api-tests-runner:latest"
  //try {
  //  sh "docker exec api_tests_runner_$env cucumber PLATFORM=$env --format html --out test-output/report.html"
  //} finally {
  //  sh "docker cp api_tests_runner_$env:/api-tests/test-output/report.html report_$env.html"
  //  sh "docker rm -rf api_tests_runner_$env"
  //}
}

def deploy(String env) {
  echo "Deploying environment... $env"
  sh "kubectl set image deployment python-greetings-$env python-greetings-$env-pod=einarsngalejs/python-greetings-app:$GIT_COMMIT"
  //sh "kubectl scale deploy python-greetings-$env --replicas=0"
  //sh "kubectl scale deploy python-greetings-$env --replicas=2"
}

def notify() {
  script {
    discordSend description: "Jenkins Pipeline Build: ${env.GIT_COMMIT}", link: env.BUILD_URL, result: currentBuild.currentResult, title: JOB_NAME, webhookURL: env.WEBHOOK_URL
  }
}
