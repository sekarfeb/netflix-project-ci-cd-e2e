pipeline {
  agent any
  tools {
    jdk 'jdk17'
    nodejs 'node16'
  }
  environment {
    SCANNER_HOME = tool 'sonar-scanner'
  }
  stages {
    stage('clean workspace') {
      steps {
        cleanWs()
      }
    }
    stage('Checkout from Git') {
      steps {
        git branch: 'main', url: 'https://github.com/sekarfeb/netflix-project-ci-cd-e2e.git'
      }
    }
    stage("Sonarqube Analysis ") {
      steps {
        withSonarQubeEnv('sonar-server') {
          sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix1 \
                    -Dsonar.projectKey=Netflix1 '''
        }
      }
    }
    stage("quality gate") {
      steps {
        script {
          waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
        }
      }
    }
    stage('Install Dependencies') {
      steps {
        sh "npm install"
      }
    }
    stage('OWASP FS SCAN') {
      steps {
        dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
        dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
      }
    }
    stage('TRIVY FS SCAN') {
      steps {
        sh "trivy fs . > trivyfs.txt"
      }
    }
    stage("Docker Build & Push") {
      steps {
        script {
          withDockerRegistry(credentialsId: 'sekar-docker', toolName: 'docker') {
            sh "docker build --build-arg TMDB_V3_API_KEY=214e59056efd2b05eed1ec3dad535e09 -t netflix-v2 ."
            sh "docker tag netflix-v2 sekarfeb/netflix-v2:latest "
            sh "docker push sekarfeb/netflix-v2:latest "
          }
        }
      }
    }
    stage("TRIVY") {
      steps {
        sh "trivy image sekarfeb/netflix-v2:latest > trivyimage.txt"
      }
    }
    stage('Deploy to container') {
      steps {
        // Stop and remove the existing container if it exists
        sh 'docker stop netflix-v2 || true' // Stop the container (if it exists)
        sh 'docker rm netflix-v2 || true' // Remove the container (if it exists)

        // Run the new container
        sh 'docker run -d --name netflix-v2 -p 8082:80 sekarfeb/netflix-v2:latest'
      }
    }
  }
}