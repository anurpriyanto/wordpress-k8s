pipeline {
    agent any
    
    environment {
        // Define environment variables
        KUBECONFIG = credentials('kubeconfig-credentials')
        GIT_REPO = 'your-git-repo-url'
        BRANCH = 'main'
        KUBE_NAMESPACE = 'wordpress'
    }
    
    stages {
        stage('Checkout') {
            steps {
                // Checkout code from repository
                git branch: "${BRANCH}", url: "${GIT_REPO}"
            }
        }
        
        stage('Validate Kubernetes Files') {
            steps {
                script {
                    // Check if kubectl is installed
                    sh 'kubectl version --client'
                    
                    // Validate Kubernetes YAML files
                    sh '''
                        for file in $(find . -name "*.yaml"); do
                            kubectl --kubeconfig=$KUBECONFIG apply --dry-run=client -f $file
                        done
                    '''
                }
            }
        }
        
        stage('Create Namespace') {
            steps {
                script {
                    sh """
                        kubectl --kubeconfig=$KUBECONFIG get namespace ${KUBE_NAMESPACE} || \
                        kubectl --kubeconfig=$KUBECONFIG create namespace ${KUBE_NAMESPACE}
                    """
                }
            }
        }
        
        stage('Deploy MySQL') {
            steps {
                script {
                    // Deploy MySQL Secret, PVC, Deployment and Service
                    sh """
                        kubectl --kubeconfig=$KUBECONFIG apply -f mysql-secret.yaml -n ${KUBE_NAMESPACE}
                        kubectl --kubeconfig=$KUBECONFIG apply -f mysql-pvc.yaml -n ${KUBE_NAMESPACE}
                        kubectl --kubeconfig=$KUBECONFIG apply -f mysql-deployment.yaml -n ${KUBE_NAMESPACE}
                        kubectl --kubeconfig=$KUBECONFIG apply -f mysql-service.yaml -n ${KUBE_NAMESPACE}
                    """
                    
                    // Wait for MySQL pod to be ready
                    sh """
                        kubectl --kubeconfig=$KUBECONFIG wait --for=condition=ready pod \
                        -l app=wordpress,tier=mysql -n ${KUBE_NAMESPACE} --timeout=300s
                    """
                }
            }
        }
        
        stage('Deploy WordPress') {
            steps {
                script {
                    // Deploy WordPress PVC, Deployment and Service
                    sh """
                        kubectl --kubeconfig=$KUBECONFIG apply -f wordpress-pvc.yaml -n ${KUBE_NAMESPACE}
                        kubectl --kubeconfig=$KUBECONFIG apply -f wordpress-deployment.yaml -n ${KUBE_NAMESPACE}
                        kubectl --kubeconfig=$KUBECONFIG apply -f wordpress-service.yaml -n ${KUBE_NAMESPACE}
                    """
                    
                    // Wait for WordPress pod to be ready
                    sh """
                        kubectl --kubeconfig=$KUBECONFIG wait --for=condition=ready pod \
                        -l app=wordpress,tier=frontend -n ${KUBE_NAMESPACE} --timeout=300s
                    """
                }
            }
        }
        
        stage('Deploy Ingress') {
            steps {
                script {
                    // Deploy Ingress configuration
                    sh """
                        kubectl --kubeconfig=$KUBECONFIG apply -f wordpress-ingress.yaml -n ${KUBE_NAMESPACE}
                    """
                }
            }
        }
        
        stage('Verify Deployment') {
            steps {
                script {
                    // Check deployment status
                    sh """
                        kubectl --kubeconfig=$KUBECONFIG get pods -n ${KUBE_NAMESPACE}
                        kubectl --kubeconfig=$KUBECONFIG get services -n ${KUBE_NAMESPACE}
                        kubectl --kubeconfig=$KUBECONFIG get ingress -n ${KUBE_NAMESPACE}
                    """
                }
            }
        }
    }
    
    post {
        success {
            echo 'Deployment successful!'
            // Add notifications here (Slack, email, etc.)
        }
        failure {
            echo 'Deployment failed!'
            // Add failure notifications and potential rollback steps
        }
        always {
            // Clean workspace
            cleanWs()
        }
    }
}
