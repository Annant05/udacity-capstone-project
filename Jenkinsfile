pipeline {
  environment {
    DEPLOY_TYPE = 'blue'
    EKS_CLUSTERNAME = 'CapstoneEKSCluster'
    registry = 'annantguptacs15/capestonenginx'
    registryCredential = 'dockerhub'
    dockerImgVar = ''
  }
  agent any
  stages {
    stage('output whoami') {
      steps {
        sh 'whoami'
      }
    }
    stage('Lint HTML') {
      steps {
        sh 'tidy -q -e ./project-src/$DEPLOY_TYPE/html/*.html'
      }
    }
    stage('Lint DockerFile') {
      steps {
        sh 'hadolint ./project-src/$DEPLOY_TYPE/Dockerfile'
      }
    }
    stage('Build Docker Image and Publish ') {
      steps {  
          dir("./project-src/${env.DEPLOY_TYPE}/"){
            script {
                dockerImgVar = docker.build registry + ":$DEPLOY_TYPE"
                docker.withRegistry('', registryCredential) {
                dockerImgVar.push()
                }
            }
        }
      }
    }
    stage('Set kubectl for the cluster') {
      steps {
        withAWS(region: 'us-east-1', credentials: 'aws-cred') {
          sh '''
                        aws eks --region us-east-1 update-kubeconfig --name $EKS_CLUSTERNAME
                    '''
        }
      }
    }
    stage('Create Kubernetes deployment') {
      steps {
        sh 'kubectl apply -f ./kubernetes/$DEPLOY_TYPE-deployment.yml'
      }
    }
    stage('Create Kubernetes sevice') {
      steps {
        sh 'kubectl apply -f ./kubernetes/deploy-service.yml'
      }
    }
    stage('Output EKS service output') {
      steps {
        sh 'kubectl get service/lb-apache-service'
      }
    }
  }
}