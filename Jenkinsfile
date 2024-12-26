pipeline {
  environment {
    KUBERNETES_NAMESPACE = 'anur-wordpress'
  }
  agent any
  stages {
    stage('Checkout Source') {
      steps {
	      checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'token-key-github', url: 'https://github.com/anurpriyanto/wordpress-k8s.git']])
      }
    }
    stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Deploy to Kubernetes using kubectl
                    sh '''
		    	kubectl apply -f deployment.yaml -n $KUBERNETES_NAMESPACE
                    '''
                }
            }
        }
  } 
}
