/* import shared library */
@Library('shared-library')_

pipeline {
  environment {
    IMAGE_NAME = "staticwebsite"
    IMAGE_TAG = "latest"
    STAGING = "spasmojo-staging"
    PRODUCTION = "spasmojo"
    PORT="5000"
  }
  agent none
  stages {
    stage('Build image') {
      agent any
      steps {
        script {
          sh 'docker build . -t spasmojo/$IMAGE_NAME:$IMAGE_TAG'
        }
      } 
    }
    stage('run container based on builded image') {
      agent any
      steps {
        script {
          sh '''
            docker run -d --name $IMAGE_NAME -p 80:$PORT -e PORT=$PORT spasmojo/$IMAGE_NAME:$IMAGE_TAG
            sleep 5
          '''
        }
      } 
    }
    stage('test image') {
      agent any
      steps {
        script {
          sh '''
            curl http://172.17.0.1 | grep -i "Dimension"
          '''
        }
      } 
    }
    stage('clean container') {
      agent any
      steps {
        script {
          sh '''
            docker stop $IMAGE_NAME
            docker rm $IMAGE_NAME
          '''
        }
      } 
    }
    stage('Push image in staging and deploy it') {
      when {
        expression { GIT_BRANCH == 'origin/master' }
      }
      agent any
      environment {
        HEROKU_API_KEY = credentials('heroku_api_key')
      }
      steps {
        script {
          sh '''
            heroku container:login
            heroku create $STAGING || echo "project already exist"
            heroku container:push -a $STAGING web
            heroku container:release -a $STAGING web
          '''
        }
      } 
    }
    stage('Push image in production and deploy it') {
      when {
        expression { GIT_BRANCH == 'origin/master' }
      }
      agent any
      environment {
        HEROKU_API_KEY = credentials('heroku_api_key')
      }
      steps {
        script {
          sh '''
            heroku container:login
            heroku create $PRODUCTION || echo "project already exist"
            heroku container:push -a $PRODUCTION web
            heroku container:release -a $PRODUCTION web
          '''
        }
      } 
    }
  }
  post {
    always {
      script {
        slackNotifier currentBuild.result
      }
    }
  }
}
