pipeline {
  agent any
  environment {
    DOCKER_IMAGE = " vxnguyen1112/cdcntt"
  }

  stages {
    stage("Test") {
      agent {
          docker {
            image 'maven:3.6.1-jdk-13-alpine'
            args '-v /root/.m2:/root/.m2'
          }
      }
      steps {
        sh "mvn test"
      }
    }
    stage("build") {
      agent { node {label 'master'}}
      environment {
        DOCKER_TAG="${GIT_BRANCH.tokenize('/').pop()}-${GIT_COMMIT.substring(0,7)}"
      }
      steps {
        sh "chmod +x mvnw"
        sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} . "
        sh "docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest"
        sh "docker image ls | grep ${DOCKER_IMAGE}"
        withCredentials([usernamePassword(credentialsId: 'docker-hub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
            sh 'echo $DOCKER_PASSWORD | docker login --username $DOCKER_USERNAME --password-stdin'
            sh "docker push ${DOCKER_IMAGE}:${DOCKER_TAG}"
            sh "docker push ${DOCKER_IMAGE}:latest"
        }

        //clean to save disk
        sh "docker image rm ${DOCKER_IMAGE}:${DOCKER_TAG}"
        sh "docker image rm ${DOCKER_IMAGE}:latest"
      }
    }

  stage("deploy") {
    steps {
        sshagent(['ssh_key']) {
            sh "ssh -o StrictHostKeyChecking=no -l root 143.198.208.97 './deployBE/deploy.sh'"
        }
    }
  }
  }
  post {
    success {
      echo "SUCCESSFUL"
    }
    failure {
      echo "FAILED"
    }
  }
}