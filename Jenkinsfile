pipeline {
  environment { // Declaration of environment variables
      DOCKER_ID = "degersti" // we will use this variable to store the docker hub account name
      DOCKER_REPO = "jenkins_devops_exam" // we will use this variable to store the docker hub repository name
      CAST_IMAGE = "casts"       
      MOVIE_IMAGE = "movies" 
      DOCKER_TAG = "v.${BUILD_ID}.0" // we will tag our images with the current build in order to increment the value by 1 with each new build
  }
  agent any // Jenkins will be able to select all available agents
  stages {
    stage(' Docker Build Casts'){ // docker build image stage
      steps {
        script {
        sh '''
            docker rm -f casts
            docker build -t $DOCKER_ID/$DOCKER_REPO:$CAST_IMAGE-$DOCKER_TAG ./cast-service
            sleep 6
        '''
        }
      }
    }
    stage(' Docker Build Movies'){ // docker build image stage
      steps {
        script {
        sh '''
            docker rm -f movies
            docker build -t $DOCKER_ID/$DOCKER_REPO:$MOVIE_IMAGE-$DOCKER_TAG ./movie-service
            sleep 6
        '''
        }
      }
    }
    stage('Docker Push'){ //we pass the built image to our docker hub account
      environment
      {
        DOCKER_PASS = credentials("DOCKER_HUB_PASS") // we retrieve  docker password from secret text called docker_hub_pass saved on jenkins
      }
      steps {
        script {
            sh '''
            docker login -u $DOCKER_ID -p $DOCKER_PASS
            docker push $DOCKER_ID/$DOCKER_REPO/$CAST_IMAGE:$DOCKER_TAG
            docker push $DOCKER_ID/$DOCKER_REPO/$MOVIE_IMAGE:$DOCKER_TAG

            '''
        }
      }
    }
  }
}
