pipeline {
  
  agent any // This can be any agent, it just needs to have Docker running on it.
  
  stages {
    
    stage('Run Trivy Scan') {
      
      // Use a Docker image that has the Docker CLI and other tools you need.
      // The 'args' part is key: it mounts the host's Docker socket.
      
      agent {
        docker {
          image 'aquasec/trivy:0.43.0'
          args '-v /var/run/docker.sock:/var/run/docker.sock -v /var/jenkins_home/workspace/sample_webapp:/workdir'
        }
      }
      
      steps {
        sh '''
                    # Now we run the Trivy command inside the Trivy container,
                    # but tell it to scan the files in the current directory.
                    trivy fs \
                      --format template \
                      --template "@contrib/html.tpl" \
                      -o /workdir/trivy-report.html \
                      /workdir

                    echo "Trivy HTML report generated at trivy-report.html"
                '''
      }
    }
  }
}
