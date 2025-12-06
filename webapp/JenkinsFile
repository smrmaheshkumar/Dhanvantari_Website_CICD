pipeline {
  agent {
    docker {
      image 'abhishekf5/maven-abhishek-docker-agent:v1'
      args '--user root -v /var/run/docker.sock:/var/run/docker.sock'
    }
  }

  environment {
    SONAR_URL = "http://35.89.216.127:9000"
    DOCKER_IMAGE = "smrmaheshkumar/dhanvantari-cicd:${BUILD_NUMBER}"
    GIT_REPO_NAME = "Dhanvantari_Website_CICD"
    GIT_USER_NAME = "smrmaheshkumar"
  }

  stages {
    stage('Clone Repository') {
      steps {
        git branch: 'main', url: 'https://github.com/smrmaheshkumar/Dhanvantari_Website_CICD.git'
      }
    }

    stage('Maven Build') {
      steps {
        sh 'cd webapp && mvn clean package'
        sh 'ls -ltr webapp/target/' // Confirm WAR file is there
      }
    }

    stage('SonarQube Analysis') {
      steps {
        withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
          sh """
            cd webapp
            mvn sonar:sonar \
              -Dsonar.projectKey=dhanvantari-web \
              -Dsonar.host.url=${SONAR_URL} \
              -Dsonar.login=${SONAR_AUTH_TOKEN}
          """
        }
      }
    }

    stage('Build & Push Docker Image') {
      steps {
        script {
          // Use webapp/ as build context so it can access Dockerfile and target/webapp.war
          sh "docker build -t ${DOCKER_IMAGE} -f webapp/Dockerfile webapp/"
          def dockerImage = docker.image("${DOCKER_IMAGE}")
          docker.withRegistry('https://index.docker.io/v1/', 'docker-cred') {
            dockerImage.push()
          }
        }
      }
    }

    stage('Update K8s Deployment File') {
      steps {
        withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
          sh """
            git config user.email "smr.maheshkumar@gmail.com"
            git config user.name "smrmaheshkumar"
            sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" Kubernetes/deployment.yml
            git add Kubernetes/deployment.yml
            git commit -m "Update image tag to ${BUILD_NUMBER}"
            git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
          """
        }
      }
    }
  }

  post {
    success {
      echo "✅ Pipeline completed successfully."
    }
    failure {
      echo "❌ Pipeline failed. Check logs above."
    }
  }
}
