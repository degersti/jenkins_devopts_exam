/*************************************************************
   File:        Jenkinsfile
   Author:      Markus Gerstenberg
   
   Description: Jenkinsfile for the DevOps exam
************************************************************/
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
    /*************************************************************
      Build Docker Container
    ************************************************************/
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
    /*************************************************************
      Push to Docker HUB
    ************************************************************/
    stage('Docker Push'){ //we pass the built image to our docker hub account
      environment
      {
        DOCKER_PASS = credentials("DOCKER_HUB_PASS") // we retrieve  docker password from secret text called docker_hub_pass saved on jenkins
      }
      steps {
        script {
            sh '''
            docker login -u $DOCKER_ID -p $DOCKER_PASS
            docker push $DOCKER_ID/$DOCKER_REPO:$CAST_IMAGE-$DOCKER_TAG
            docker push $DOCKER_ID/$DOCKER_REPO:$MOVIE_IMAGE-$DOCKER_TAG

            '''
        }
      }
    }
    /*************************************************************
      Create Secrets in Kubernetes
    ************************************************************/
    stage('Prepare Kubernetes') {
      environment {
        KUBECONFIG = credentials('config')
      }
      steps {
        script {
          withCredentials([
            string(credentialsId: 'MOVIE_DB_PASSWORD', variable: 'MOVIE_PASS'),
            string(credentialsId: 'CAST_DB_PASSWORD', variable: 'CAST_PASS')
          ]) {
            sh '''
              set -eu

              for ns in dev qa staging prod; do
                  kubectl create namespace "$ns" \
                  --dry-run=client -o yaml | kubectl apply -f -

                kubectl create secret generic movie-db-secret \
                  --namespace "$ns" \
                  --from-literal=postgres-password="$MOVIE_PASS" \
                  --from-literal=database-uri="postgresql://movie_db_username:${MOVIE_PASS}@movie-db:5432/movie_db_dev" \
                  --dry-run=client -o yaml | kubectl apply -f -

                kubectl create secret generic cast-db-secret \
                  --namespace "$ns" \
                  --from-literal=postgres-password="$CAST_PASS" \
                  --from-literal=database-uri="postgresql://cast_db_username:${CAST_PASS}@cast-db:5432/cast_db_dev" \
                  --dry-run=client -o yaml | kubectl apply -f -
                done
            '''
          }
        }
      }
    }
    /*************************************************************
      Deoloy in Namespaces dev, qa, staging and prod
    ************************************************************/
    stage('Deployment in dev') {
      environment {
          KUBECONFIG = credentials('config')
      }
      steps {
        script {
          sh '''
            helm upgrade --install movie-app ./charts \
              --namespace dev \
              --create-namespace \
              --set movie.image.repository=$DOCKER_ID/$DOCKER_REPO \
              --set movie.image.tag=$MOVIE_IMAGE-$DOCKER_TAG \
              --set movie.image.pullPolicy=IfNotPresent \
              --set cast.image.repository=$DOCKER_ID/$DOCKER_REPO \
              --set cast.image.tag=$CAST_IMAGE-$DOCKER_TAG \
              --set cast.image.pullPolicy=IfNotPresent \
              --wait \
              --timeout 5m
          '''
        }
      }
    }
    stage('Deployment in qa') {
      environment {
          KUBECONFIG = credentials('config')
      }
      steps {
        script {
          sh '''
            helm upgrade --install movie-app ./charts \
              --namespace qa \
              --create-namespace \
              --set movie.image.repository=$DOCKER_ID/$DOCKER_REPO \
              --set movie.image.tag=$MOVIE_IMAGE-$DOCKER_TAG \
              --set movie.image.pullPolicy=IfNotPresent \
              --set cast.image.repository=$DOCKER_ID/$DOCKER_REPO \
              --set cast.image.tag=$CAST_IMAGE-$DOCKER_TAG \
              --set cast.image.pullPolicy=IfNotPresent \
              --wait \
              --timeout 5m
          '''
        }
      }
    }
    stage('Deployment in staging') {
      environment {
          KUBECONFIG = credentials('config')
      }
      steps {
        script {
          sh '''
            helm upgrade --install movie-app ./charts \
              --namespace staging \
              --create-namespace \
              --set movie.image.repository=$DOCKER_ID/$DOCKER_REPO \
              --set movie.image.tag=$MOVIE_IMAGE-$DOCKER_TAG \
              --set movie.image.pullPolicy=IfNotPresent \
              --set cast.image.repository=$DOCKER_ID/$DOCKER_REPO \
              --set cast.image.tag=$CAST_IMAGE-$DOCKER_TAG \
              --set cast.image.pullPolicy=IfNotPresent \
              --wait \
              --timeout 5m
          '''
        }
      }
    }
  }
}

