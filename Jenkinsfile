pipeline {
  agent {
    label 'jnlp-himem'
  }

  environment {
    IMAGE_NAME = "eosc-node-endpoint-service"
    REGISTRY = "${DOCKER_ARTIFACT_REGISTRY}"
    DOCKER_TAG = "${GIT_COMMIT}"
    DOCKER_IMAGE = "${DOCKER_ARTIFACT_REGISTRY}/${IMAGE_NAME}:${GIT_COMMIT}"
  }

  stages {
    stage('Build & Test') {
      steps {
        withMaven {
          sh './mvnw clean verify'
        }
      }
    }
    stage('Run Sonar Scan') {
      steps {
        withSonarQubeEnv('cessda-sonar') {
          withMaven {
            sh "./mvnw sonar:sonar -Pci"
          }
        }
      }
      when { branch 'main' }
    }
    stage('Get Sonar Quality Gate') {
      steps {
        timeout(time: 1, unit: 'HOURS') {
          waitForQualityGate abortPipeline: true
        }
      }
      when { branch 'main' }
    }
    stage('Build Image') {
      steps {
        withMaven {
          sh "./mvnw -pl eosc-node-endpoint-service spring-boot:build-image-no-fork -Dspring-boot.build-image.imageName=${DOCKER_IMAGE}"
        }
      }
      when { branch 'main' }
    }

    stage('Upload Image') {
      steps {
        sh "gcloud auth configure-docker ${ARTIFACT_REGISTRY_HOST}"
        sh "docker push ${DOCKER_IMAGE}"
      }
      when { branch 'main' }
    }

    stage('Deploy Node Endpoint') {
      steps {
        build job: 'cessda.resource-catalogue.deploy/main', parameters: [string(name: 'NODE_ENDPOINT_IMAGE_TAG', value: GIT_COMMIT)], wait: false
      }
      when { branch 'main' }
    }
  }
}
