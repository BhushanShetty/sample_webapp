pipeline {
    // Tell Jenkins this pipeline MUST run on an agent with the 'windows-docker' label
    agent {
        label 'windows-docker'
    }

    stages {
        stage('Run Trivy Scan on Windows') {
            steps {
                // Use 'bat' for Windows batch commands instead of 'sh'
                // Use %cd% for the current directory instead of $(pwd)
                bat '''
                    echo "Starting Trivy Scan on a Windows Agent..."

                    docker run --rm -v "%cd%:/workdir" aquasec/trivy:0.43.0 --format template --template "@contrib/html.tpl" -o /workdir/trivy-report.html .

                    echo "Trivy HTML report generated at trivy-report.html"
                '''
            }
        }
    }
}
