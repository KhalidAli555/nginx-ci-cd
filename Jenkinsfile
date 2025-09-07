pipeline {
    agent any
    environment {
        DEV_NAMESPACE = "dev"
        PROD_NAMESPACE = "prod"
        DB_NAMESPACE = "database"
        JENKINS_NAMESPACE = "jenkins"
        KUBECTL = "/tmp/kubectl"
    }
    stages {
        stage('Checkout') {
            steps {
                echo "Cloning GitHub repository..."
                git url: 'https://github.com/KhalidAli555/nginx-ci-cd.git', branch: 'main'
            }
        }

        stage('Deploy Database') {
            steps {
                echo "Deploying Database..."
                sh "${KUBECTL} apply -f database/deployment.yml -n ${DB_NAMESPACE}"
                sh "${KUBECTL} apply -f database/service.yml -n ${DB_NAMESPACE}"
                sh "${KUBECTL} apply -f database/pvc.yml -n ${DB_NAMESPACE}"
                sh "${KUBECTL} apply -f database/secret.yml -n ${DB_NAMESPACE}"
                sh "${KUBECTL} get pods -n ${DB_NAMESPACE}"
            }
        }

        stage('Deploy Jenkins') {
            steps {
                echo "Deploying Jenkins..."
                sh "${KUBECTL} apply -f jenkins/jenkins-deployment.yml -n ${JENKINS_NAMESPACE}"
                sh "${KUBECTL} apply -f jenkins/jenkins-service.yml -n ${JENKINS_NAMESPACE}"
                sh "${KUBECTL} apply -f jenkins/jenkins-pvc.yml -n ${JENKINS_NAMESPACE}"
                sh "${KUBECTL} apply -f jenkins/jenkins-egress.yaml -n ${JENKINS_NAMESPACE}"
                sh "${KUBECTL} get pods -n ${JENKINS_NAMESPACE}"
            }
        }

        stage('Deploy to Dev') {
            steps {
                echo "Deploying Nginx App to Dev..."
                sh "${KUBECTL} apply -f dev/deployment.yml -n ${DEV_NAMESPACE}"
                sh "${KUBECTL} apply -f dev/service.yml -n ${DEV_NAMESPACE}"
                sh "${KUBECTL} apply -f dev/pvc.yml -n ${DEV_NAMESPACE}"
                sh "${KUBECTL} get pods -n ${DEV_NAMESPACE}"
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
                sh "${KUBECTL} apply -f prod/deployment.yml -n ${PROD_NAMESPACE}"
                sh "${KUBECTL} apply -f prod/service.yml -n ${PROD_NAMESPACE}"
                sh "${KUBECTL} apply -f prod/pvc.yml -n ${PROD_NAMESPACE}"
                sh "${KUBECTL} get pods -n ${PROD_NAMESPACE}"
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully!"
        }
        failure {
            echo "Pipeline failed!"
        }
    }
}
