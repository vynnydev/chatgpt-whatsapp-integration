pipeline {
  agent any

  stages {

    stage ('Build Docker Image') {
      steps {
        script {
          dockerapp = docker.build("tfk8scloud/purchases-srv:${env.BUILD_ID}", './packages/services/purchases/Dockerfile .')
        }
      }
    }

    stage ('Push Docker Image') {
      steps {
        script {
          docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
            dockerapp.push('latest')
            dockerapp.push("${env.BUILD_ID}")
          }
        }
      }
    }

    stage('Deploy Kubernetes') {
      environment {
        tag_version = "${env.BUILD_ID}"
      }
      steps {
        withKubeconfig([credentialsId: 'kubeconfig']) {
          sh 'sed -i "s/{{tag}}/$tag_version/g" ./k8s/services/purchases/deployment.yaml'
          sh 'kubectl apply -f ./k8s/services/purchases/pod.yaml'
        }
      }
    }
  }
}