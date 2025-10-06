pipeline {
    // This top-level agent can be any basic agent.
    agent any

    environment {
        SONAR_TOKEN = credentials('sonar-token')
        DOCKER_IMAGE = "my-python-app:latest"
        TRIVY_IMAGE = 'aquasec/trivy:0.43.0' 
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/BhushanShetty/sample_webapp/'
            }
        }
        
        // This stage runs on the basic agent, as it only needs Python.
        stage('Setup, Test & SonarQube Scan') {
            steps {
                sh '''
                python -m venv venv
                source venv/bin/activate
                pip install -r requirements.txt
                pytest tests/
                '''
                withSonarQubeEnv('sonarqube') {
                    sh '''
                    source venv/bin/activate
                    sonar-scanner -Dsonar.login=$SONAR_TOKEN
                    '''
                }
            }
        }

        // =====================================================================
        // CRITICAL SECTION: This is the part that solves your "docker: not found" error
        // =====================================================================
        stage('Security Scans & Docker Build') {
            // THIS IS THE FIX: It tells Jenkins to run all nested stages
            // inside a container created from the 'docker:24.0.5' image.
            // This container has the 'docker' command available.
            agent {
                docker { image 'docker:24.0.5' }
            }
            
            stages {
                stage('Trivy Filesystem Scan') {
                    steps {
                        // Now this 'docker run' command will succeed because we are inside a Docker-enabled agent.
                        sh """
                        docker run --rm -v \$(pwd):/workdir ${TRIVY_IMAGE} --format template --template "@contrib/html.tpl" -o trivy-fs-report.html .
                        """
                    }
                    post {
                        always {
                            archiveArtifacts artifacts: 'trivy-fs-report.html', allowEmptyArchive: true
                        }
                    }
                }

                stage('Docker Build') {
                    steps {
                        sh "docker build -t ${DOCKER_IMAGE} ."
                    }
                }

                stage('Trivy Image Scan') {
                    steps {
                        sh """
                        docker run --rm -v /var/run/docker.sock:/var/run/docker.sock ${TRIVY_IMAGE} image --format template --template "@contrib/html.tpl" -o trivy-image-report.html --exit-code 1 --severity HIGH,CRITICAL ${DOCKER_IMAGE}
                        """
                    }
                    post {
                        always {
                             publishHTML(target: [
                                allowMissing: false,
                                alwaysLinkToLastBuild: true,
                                keepAll: true,
                                reportDir: '.',
                                reportFiles: 'trivy-image-report.html',
                                reportName: 'Trivy Image Vulnerability Report'
                            ])
                        }
                    }
                }
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
            cleanWs()
        }
    }
}
