pipeline {
    agent any
    environment {
        SONAR_TOKEN = credentials('sonar-token')
        DOCKER_IMAGE = "my-python-app:latest"
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/yourusername/my-python-app.git'
            }
        }

        stage('Setup Python Environment') {
            steps {
                sh '''
                python -m venv venv
                source venv/bin/activate
                pip install -r requirements.txt
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh '''
                source venv/bin/activate
                pytest tests/
                '''
            }
        }

        stage('SonarQube Scan') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh '''
                    source venv/bin/activate
                    sonar-scanner -Dsonar.login=$SONAR_TOKEN
                    '''
                }
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE} ."
                }
            }
        }

        stage('Trivy Scan') {
            steps {
                sh "trivy image ${DOCKER_IMAGE}"
            }
        }

        stage('Docker Scout Scan') {
            steps {
                sh "docker scout analyze ${DOCKER_IMAGE}"
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                kubectl apply -f k8s/deployment.yaml
                kubectl apply -f k8s/service.yaml
                '''
            }
        }
    }
    post {
        always {
            echo 'Pipeline completed.'
        }
    }
}
