// anchore plugin for jenkins: https://www.jenkins.io/doc/pipeline/steps/anchore-container-scanner/

pipeline {
  environment {
    // "REGISTRY" isn't required if we're using docker hub, I'm leaving it here in case you want to use a different registry
    // REGISTRY = 'registry.hub.docker.com'
    // you need a credential named 'docker-hub' with your DockerID/password to push images
    CREDENTIAL = "docker-hub"
    DOCKER_HUB = credentials("$CREDENTIAL")
    REPOSITORY = "${DOCKER_HUB_USR}/anchore-jenkins-pipeline-demo"
    TAG1 = "image1-${BUILD_NUMBER}"
    TAG2 = "image2-${BUILD_NUMBER}"
    IMAGELINE1 = "${REPOSITORY}:${TAG1} Dockerfile-1"
    IMAGELINE2 = "${REPOSITORY}:${TAG2} Dockerfile-2"
} // end environment 
  agent any
  stages {
    stage('Checkout SCM') {
      steps {
        checkout scm
      } // end steps
    } // end stage "checkout scm"
    stage('Build and push Image 1') {
      steps {
        script {
          sh """
            docker login -u ${DOCKER_HUB_USR} -p ${DOCKER_HUB_PSW}
            docker build -t ${REPOSITORY}:${TAG1} -f ./Dockerfile-1 .
            docker push ${REPOSITORY}:${TAG1}
          """
        } // end script
      } // end steps
    } // end stage "build image and push to registry"
    stage('Analyze with Anchore plugin') {
      steps {
        writeFile file: 'anchore_images-1', text: IMAGELINE1
        script {
          try {
            // forceAnalyze is a good idea since we're passing a Dockerfile with the image
            anchore name: 'anchore_images-1', forceAnalyze: 'true', engineRetries: '900'
          } catch (err) {
            // if scan fails, clean up (delete the image) and fail the build
            sh 'docker rmi ${REPOSITORY}:${TAG}'
            sh 'exit 1'
          } // end try
        } // end script 
      } // end steps
    } // end stage "analyze with anchore plugin"
    stage('Re-tag as prod and push stable image to registry') {
      steps {
        script {
          docker.withRegistry('', CREDENTIAL) {
            dockerImage.push('prod') 
            // dockerImage.push takes the argument as a new tag for the image before pushing
          }
        } // end script 
      } // end steps
    } // end stage "retag as prod"
    stage('Clean up') {
      // if we succuessfully pushed the :prod tag than we don't need the $BUILD_ID tag anymore
      steps {
        sh 'docker rmi ${REPOSITORY}${TAG} ${REPOSITORY}:prod'
      } // end steps
    } // end stage "clean up"
  } // end stages
} // end pipeline 
