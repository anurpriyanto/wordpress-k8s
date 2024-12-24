pipeline {
  environment {
    KUBERNETES_NAMESPACE = 'dedi-wordpress'
  }
  agent any
  stages {
    stage('Checkout Source') {
      steps {
	      checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'token-key-github', url: 'https://github.com/deditriyadibarnawan69/dedi-wordpress.git']])
      }
    }
    stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Deploy to Kubernetes using kubectl
                    sh '''
                        kubectl create namespace $KUBERNETES_NAMESPACE
                        kubectl apply -f manifest-wordpress.yaml -n $KUBERNETES_NAMESPACE
                    '''
                }
            }
        }
  } 
}
