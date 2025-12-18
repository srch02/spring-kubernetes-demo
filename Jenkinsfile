pipeline {
    agent any

    environment {
        KUBE_NAMESPACE = 'devops'
    }

    stages {
        // 1. Créer le namespace
        stage('Create Namespace') {
            steps {
                sh """
                    kubectl create namespace ${KUBE_NAMESPACE} --dry-run=client -o yaml | kubectl apply -f -
                """
            }
        }

        // 2. Déployer MySQL (fichier YAML inline)
        stage('Deploy MySQL') {
            steps {
                sh """
                    cat <<EOF | kubectl apply -n ${KUBE_NAMESPACE} -f -
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/data/mysql"
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: root123
        - name: MYSQL_DATABASE
          value: springdb
        ports:
        - containerPort: 3306
        volumeMounts:
        - name: mysql-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-storage
        persistentVolumeClaim:
          claimName: mysql-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: mysql-service
spec:
  selector:
    app: mysql
  ports:
    - port: 3306
      targetPort: 3306
  type: ClusterIP
EOF
                """
                sh "kubectl rollout status deployment/mysql -n ${KUBE_NAMESPACE} --timeout=120s"
            }
        }

        // 3. Déployer Spring Boot (fichier YAML inline)
        stage('Deploy Spring Boot') {
            steps {
                sh """
                    cat <<EOF | kubectl apply -n ${KUBE_NAMESPACE} -f -
apiVersion: v1
kind: ConfigMap
metadata:
  name: spring-config
data:
  SPRING_DATASOURCE_URL: jdbc:mysql://mysql-service:3306/springdb
---
apiVersion: v1
kind: Secret
metadata:
  name: spring-secret
type: Opaque
data:
  SPRING_DATASOURCE_USERNAME: c3ByaW5n        # spring
  SPRING_DATASOURCE_PASSWORD: c3ByaW5nMTIz    # spring123
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: spring-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: spring-app
  template:
    metadata:
      labels:
        app: spring-app
    spec:
      containers:
      - name: spring-app
        image: gcr.io/google-samples/hello-app:1.0   # Image de test (pas Spring)
        ports:
        - containerPort: 8080
        envFrom:
        - configMapRef:
            name: spring-config
        - secretRef:
            name: spring-secret
---
apiVersion: v1
kind: Service
metadata:
  name: spring-service
spec:
  selector:
    app: spring-app
  ports:
    - port: 8080
      targetPort: 8080
      nodePort: 30080
  type: NodePort
EOF
                """
                sh "kubectl rollout status deployment/spring-app -n ${KUBE_NAMESPACE} --timeout=120s"
            }
        }

        // 4. Vérification
        stage('Verification') {
            steps {
                sh "kubectl get pods,svc -n ${KUBE_NAMESPACE}"
                sh "minikube service spring-service -n ${KUBE_NAMESPACE} --url || true"
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline réussie ! Application déployée sur Kubernetes.'
        }
        failure {
            echo '❌ Pipeline échouée.'
        }
    }
}