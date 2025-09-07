pipeline {
    agent any
    environment {
        DEV_NAMESPACE = "dev"
        PROD_NAMESPACE = "prod"
        DB_NAMESPACE = "database"
        JENKINS_NAMESPACE = "jenkins"
    }
    stages {
        stage('Setup Kubectl') {
            steps {
                echo "Installing kubectl..."
                sh '''
                  curl -LO "https://dl.k8s.io/release/v1.29.0/bin/linux/amd64/kubectl"
                  chmod +x kubectl
                  sudo mv kubectl /usr/local/bin/
                  kubectl version --client
                '''
            }
        }

        stage('Checkout') {
            steps {
                echo "Cloning GitHub repository..."
                git url: 'https://github.com/KhalidAli555/nginx-ci-cd.git', branch: 'main'
            }
        }

        stage('Deploy Database') {
            steps {
                echo "Deploying Database..."
                sh "kubectl apply -f database/deployment.yml -n ${DB_NAMESPACE}"
                sh "kubectl apply -f database/service.yml -n ${DB_NAMESPACE}"
                sh "kubectl apply -f database/pvc.yml -n ${DB_NAMESPACE}"
                sh "kubectl apply -f database/secret.yml -n ${DB_NAMESPACE}"
                sh "kubectl get pods -n ${DB_NAMESPACE}"
            }
        }

        stage('Deploy Jenkins') {
            steps {
                echo "Deploying Jenkins..."
                sh "kubectl apply -f jenkins/jenkins-deployment.yml -n ${JENKINS_NAMESPACE}"
                sh "kubectl apply -f jenkins/jenkins-service.yml -n ${JENKINS_NAMESPACE}"
                sh "kubectl apply -f jenkins/jenkins-pvc.yml -n ${JENKINS_NAMESPACE}"
                sh "kubectl apply -f jenkins/jenkins-egress.yaml -n ${JENKINS_NAMESPACE}"
                sh "kubectl get pods -n ${JENKINS_NAMESPACE}"
            }
        }

        stage('Deploy to Dev') {
            steps {
                echo "Deploying Nginx App to Dev..."
                sh "kubectl apply -f dev/deployment.yml -n ${DEV_NAMESPACE}"
                sh "kubectl apply -f dev/service.yml -n ${DEV_NAMESPACE}"
                sh "kubectl apply -f dev/pvc.yml -n ${DEV_NAMESPACE}"
                sh "kubectl get pods -n ${DEV_NAMESPACE}"
            }
        }

        stage('Approval before Prod') {
            steps {
                input message: "Approve deployment to PROD?", ok: "Deploy"
            }
        }

        stage('Deploy to Prod') {
            steps {
                echo "Deploying Nginx App to Prod..."
                sh "kubectl apply -f prod/deployment.yml -n ${PROD_NAMESPACE}"
                sh "kubectl apply -f prod/service.yml -n ${PROD_NAMESPACE}"
                sh "kubectl apply -f prod/pvc.yml -n ${PROD_NAMESPACE}"
