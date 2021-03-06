pipeline {
  agent any

  stages {

    stage('Build') {
      environment {
        DOCKERHUB_CREDS = credentials('dockerhub')
      }
      steps {
          // Build new image
          sh "docker build -t invaleed/hello-docker:${env.GIT_COMMIT} ."
          // Publish new image
          sh "docker login --username $DOCKERHUB_CREDS_USR --password $DOCKERHUB_CREDS_PSW && docker push invaleed/hello-docker:${env.GIT_COMMIT}"
      }
    }

    stage('Deploy Dev') {
      steps {
          sh "rm -rf hello-docker-deploy"  
          sh "git clone https://github.com/invaleed/hello-docker-deploy.git"
          sh "git config --global user.name 'Ramadoni'"
          sh "git config --global user.email 'ramadoni.ashudi@gmail.com'"

          dir("hello-docker-deploy") {
            sh "cd ./dev && kustomize edit set image invaleed/hello-docker:${env.GIT_COMMIT}"
            sh "git commit -am 'Publish new version' && git push || echo 'no changes'"
        }
      }
    }

    stage('Deploy to Prod') {
      steps {
        input message:'Approve deployment to PRD?'
          dir("hello-docker-deploy") {
            sh "cd ./prd && kustomize edit set image invaleed/hello-docker:${env.GIT_COMMIT}"
            sh "git commit -am 'Publish new version' && git push || echo 'no changes'"
          }
      }
      
      post {
        success {
          slackSend (color: '#00FF00', message: "Deployment Completed Successfully: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        }
        failure {
          slackSend (color: '#FF0000', message: "Deployment Failed !: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        }
      }
    }
  }
}
