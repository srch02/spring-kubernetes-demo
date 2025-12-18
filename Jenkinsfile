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
                sh 'ls -la'
            }
        }
        
        stage('Deploy MySQL (Simulated)') {
            steps {
                sh """
                    kubectl apply -f mysql-deployment.yaml -n ${KUBE_NAMESPACE}
                    echo "✅ MySQL deployment submitted"
                """
            }
        }
        
        stage('Deploy Spring Boot (Simulated)') {
            steps {
                sh """
                    kubectl apply -f spring-deployment.yaml -n ${KUBE_NAMESPACE}
                    echo "✅ Spring Boot deployment submitted"
                """
            }
        }
        
        stage('Verify Deployment') {
            steps {
                sh "sleep 10"  # Wait for pods to start
                sh "kubectl get pods,svc -n ${KUBE_NAMESPACE}"
                sh "echo '=== Workshop Pipeline SUCCESS ==='"
                sh "echo 'Atelier 4: Kubernetes + Jenkins CI/CD Completed!'"
            }
        }
    }
    
    post {
        always {
            echo "=== Final Status ==="
            sh "kubectl get all -n ${KUBE_NAMESPACE} || true"
        }
        success {
            echo '🎉 BRAVO! Atelier 4 Kubernetes terminé avec succès!'
            echo 'Vous avez réalisé:'
            echo '1. Configuration Jenkins → Kubernetes ✓'
            echo '2. Pipeline CI/CD complet ✓'
            echo '3. Déploiement automatique sur cluster ✓'
        }
    }
}