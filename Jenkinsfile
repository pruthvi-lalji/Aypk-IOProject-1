pipeline {
  environment {
    PROJECT_DIR = "/app"
    CONTAINER_NAME = "aypk"
    DOCKER_ACCOUNT = "oabuoun"
    REGISTRY = "$DOCKER_ACCOUNT" + "/" + "$CONTAINER_NAME"
    IMAGE_NAME = "$REGISTRY" + ":" + "$BUILD_NUMBER"
    REGISTRY_CREDENTIALS = "docker_auth"
    DOCKER_IMAGE = ''
  }

  agent any

  options {
    skipStagesAfterUnstable()
  }

  stages {
    stage('Clone from Git') {
      steps {
        checkout scm
      }
    }

    stage('Build-Image') {
      steps{
        script {
          DOCKER_IMAGE = docker.build IMAGE_NAME
        }
      }
    }

    stage('Test') {
    	steps {
        script {
          sh '''
            docker run --rm -v $PWD/test-results:/reports --workdir $PROJECT_DIR --name $CONTAINER_NAME $IMAGE_NAME pytest -v --junitxml=/reports/results.xml
          '''
        }
      }
    	post {
    			always {
    					junit testResults: '**/test-results/*.xml'
    			}
    	}
    }

    stage('Deploy Image') {
      steps{
        script {
          docker.withRegistry( '', REGISTRY_CREDENTIALS ) {
            DOCKER_IMAGE.push()
          }
        }
      }
    }

    stage('Remove Unused docker image') {
      steps{
        sh "docker rmi $IMAGE_NAME"
      }
    }
  }
}


