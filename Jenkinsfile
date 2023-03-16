pipeline {
  agent {
    kubernetes {
      label 'agent'
      defaultContainer 'jnlp'
    }
  }
  stages {
    stage('Clone Git Repo') {
      steps {
        container('jnlp') {
        }
      }
    }
    stage('Build and Push Docker Image') {
      steps {
        container('docker-build') {
          withCredentials([usernamePassword(credentialsId: 'docker-login-creds', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
            sh "docker build -t grivardan/laravel:${GIT_COMMIT} ."
            sh 'docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD'
            sh "docker push grivardan/laravel:${GIT_COMMIT}"
          }
        }
      }
    }
    stage('Generate Kubernetes manifest') {
      steps {
        script {
          def deploymentTemplate = readFile('deployment.yaml.template')
          def deploymentManifest = deploymentTemplate.replace('IMAGE_NAME_BACK', "grivardan/laravel:${GIT_COMMIT}")
          writeFile file: 'deployment-back.yaml', text: deploymentManifest
        }
      }
    }
    stage('Deploy to Kubernetes') {
      steps {
        container('kubectl') {
          sh 'kubectl apply -f deployment-back.yaml'
        }
      }
    }
  }
}
