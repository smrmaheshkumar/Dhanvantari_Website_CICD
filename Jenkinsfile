pipeline {
  agent {
    docker {
      image 'abhishekf5/maven-abhishek-docker-agent:v1'
      args '--user root -v /var/run/docker.sock:/var/run/docker.sock'
    }
  }

  environment {
    SONAR_URL     = "http://35.89.216.127:9000"
    DOCKER_IMAGE  = "smrmaheshkumar/dhanvantari-cicd:${BUILD_NUMBER}"
    GIT_REPO_NAME = "Dhanvantari_Website_CICD"
    GIT_USER_NAME = "smrmaheshkumar"
  }

  stages {

    stage('Clone Repository - prod branch') {
      steps {
        git branch: 'prod', url: 'https://github.com/smrmaheshkumar/Dhanvantari_Website_CICD.git'
      }
    }

    stage('Build Maven Package') {
      steps {
        sh 'cd webapp && mvn clean package'
        sh 'ls -ltr webapp/target/' // confirm webapp.war exists
      }
    }

    stage('SonarQube Analysis') {
      steps {
        withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
          sh """
            cd webapp
            mvn sonar:sonar \\
              -Dsonar.projectKey=dhanvantari-web \\
              -Dsonar.host.url=${SONAR_URL} \\
              -Dsonar.login=${SONAR_AUTH_TOKEN}
          """
        }
      }
    }

    stage('Build and Push Docker Image') {
      steps {
        script {
          sh "docker build -t ${DOCKER_IMAGE} -f webapp/Dockerfile webapp/"
          def dockerImage = docker.image("${DOCKER_IMAGE}")
          docker.withRegistry('https://index.docker.io/v1/', 'docker-cred') {
            dockerImage.push()
          }
        }
      }
    }
  }

  post {
    success {
      echo "✅ CI Pipeline completed successfully!"
    }
    failure {
      echo "❌ CI Pipeline failed. Check logs above."
    }
  }
}
