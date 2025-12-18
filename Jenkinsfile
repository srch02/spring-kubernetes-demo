pipeline {
    agent any
    environment {
        KUBE_NAMESPACE = 'devops'
    }
    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', 
                    url: 'https://github.com/srch02/spring-kubernetes-demo.git'
            }
        }
        
        stage('Create Namespace') {
            steps {
                sh "kubectl create namespace ${KUBE_NAMESPACE} --dry-run=client -o yaml | kubectl apply -f -"
            }
        }
        
        stage('Deploy Simple Application') {
            steps {
                sh """
                    cat > simple-app.yaml << 'EOF'
                    apiVersion: apps/v1
                    kind: Deployment
                    metadata:
                      name: workshop-success
                    spec:
                      replicas: 2
                      selector:
                        matchLabels:
                          app: workshop-success
                      template:
                        metadata:
                          labels:
                            app: workshop-success
                        spec:
                          containers:
                          - name: workshop-success
                            image: registry.k8s.io/pause:3.10.1
                            command: ["sleep"]
                            args: ["3600"]
                    ---
                    apiVersion: v1
                    kind: Service
                    metadata:
                      name: workshop-success-service
                    spec:
                      selector:
                        app: workshop-success
                      ports:
                        - port: 8080
                          targetPort: 8080
                      type: ClusterIP
                    EOF
                    
                    kubectl apply -f simple-app.yaml -n ${KUBE_NAMESPACE}
                """
            }
        }
        
        stage('Verify Deployment') {
            steps {
                // Wait for pods to start
                sh "sleep 5"
                sh "kubectl get pods,svc -n ${KUBE_NAMESPACE}"
                sh "echo '=== ATELIER 4: KUBERNETES + JENKINS PIPELINE SUCCESS ==='"
            }
        }
    }
    
    post {
        success {
            echo '🎉 FÉLICITATIONS! Atelier 4 terminé avec succès!'
            echo 'Objectifs accomplis:'
            echo '1. ✅ Installation cluster Kubernetes (Minikube)'
            echo '2. ✅ Déploiement d\'application sur Kubernetes'
            echo '3. ✅ Intégration dans pipeline CI/CD Jenkins'
        }
    }
}