pipeline {
    agent any
    environment {
        SONAR_TOKEN = credentials('sonar-token')
        DOCKER_IMAGE = "my-python-app:latest"
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/BhushanShetty/sample_webapp/'
            }
        }

        // CORRECTED: Combined sh commands to ensure venv is used correctly.
        stage('Setup Python and Install Dependencies') {
            steps {
                sh '''
                python -m venv venv
                source venv/bin/activate
                pip install -r requirements.txt
                '''
            }
        }

        // CORRECTED: Activate venv in the same shell as the command.
        stage('Run Tests') {
            steps {
                sh '''
                source venv/bin/activate
                pytest tests/
                '''
            }
        }

        // CORRECTED: Activate venv in the same shell as the command.
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

        // MODIFIED: Trivy Scan stage to generate and archive a report.
        stage('Trivy Scan & Report') {
            steps {
                // This command scans the image, fails the build if HIGH or CRITICAL vulnerabilities are found,
                // and generates an HTML report regardless.
                sh """
                trivy image --format template --template "@/usr/local/share/trivy/templates/html.tpl" -o trivy-report.html --exit-code 1 --severity HIGH,CRITICAL ${DOCKER_IMAGE}
                """
            }
            post {
                always {
                    // Archive the report so it's available for download from the build page.
                    archiveArtifacts artifacts: 'trivy-report.html', allowEmptyArchive: true
                }
            }
        }

        stage('Docker Scout Scan') {
            steps {
                sh "docker scout cves ${DOCKER_IMAGE}" // 'analyze' is deprecated, using 'cves'
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
            // Clean up the virtual environment
            sh 'rm -rf venv'
        }
    }
}
