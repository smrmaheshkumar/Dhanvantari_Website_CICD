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
    NEXUS_IMAGE   = "52.43.124.187:5000/dhanvantari-cicd:${BUILD_NUMBER}"
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
        sh 'ls -ltr webapp/target/'
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

    stage('Build and Push Docker Images') {
      steps {
        script {
          // Build Docker image
          sh "docker build -t ${DOCKER_IMAGE} -f webapp/Dockerfile webapp/"

          // Push to Docker Hub
          docker.withRegistry('https://index.docker.io/v1/', 'docker-cred') {
            docker.image("${DOCKER_IMAGE}").push()
          }

          // Tag for Nexus and push
          sh "docker tag ${DOCKER_IMAGE} ${NEXUS_IMAGE}"

          withCredentials([usernamePassword(credentialsId: 'nexus-docker-cred', usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
            sh """
              echo $NEXUS_PASS | docker login 52.43.124.187:5000 --username $NEXUS_USER --password-stdin
              docker push ${NEXUS_IMAGE}
            """
          }
        }
      }
    }
  }

  post {
    success {
      echo "✅ Pipeline executed successfully!"
    }
    failure {
      echo "❌ Pipeline failed. Check the above logs."
    }
  }
}
