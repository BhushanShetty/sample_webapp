pipeline {
    // Run the main part of the pipeline on a basic agent.
    agent any

    environment {
        SONAR_TOKEN = credentials('sonar-token')
        DOCKER_IMAGE = "my-python-app:latest"
        // Define the Trivy image to use, making it easy to update
        TRIVY_IMAGE = 'aquasec/trivy:0.43.0' 
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/BhushanShetty/sample_webapp/'
            }
        }
        
        // This stage does not need Docker, so it can run on any agent.
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

        // --- Stages requiring Docker ---
        // We will define a specific agent for all stages that need the Docker CLI.
        
        stage('Security Scans & Build') {
            // This agent directive tells Jenkins to run these steps inside a Docker container
            // that has the Docker CLI available.
            agent {
                docker { image 'docker:24.0.5' }
            }
            stages {
                stage('Trivy Filesystem Scan') {
                    steps {
                        // This command runs the trivy container, mounts the current workspace,
                        // scans the source code for misconfigurations, secrets, etc.
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
                        // Now we scan the image we just built for OS and library vulnerabilities.
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

                stage('Docker Scout Scan') {
                    steps {
                         sh "docker scout cves ${DOCKER_IMAGE}"
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
            cleanWs() // Deletes the workspace at the end of the build
        }
    }
}
