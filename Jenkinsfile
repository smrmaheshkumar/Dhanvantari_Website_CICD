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
          sh "docker build -t ${DOCKER_IMAGE} -f webapp/Dockerfile webapp/"

          docker.withRegistry('https://index.docker.io/v1/', 'docker-cred') {
            docker.image("${DOCKER_IMAGE}").push()
          }

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

    stage('Update Deployment') {
      steps {
        withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
          script {
            dir('Kubernetes') {
              sh '''
                git reset --hard HEAD
                git clean -fdx
                git pull origin main
              '''

              // Replace image tag
              sh "sed -i 's/replaceImageTag/${BUILD_NUMBER}/g' deployment.yml"

              // Configure Git
              sh '''
                git config user.email "smr.maheshkumar@gmail.com"
                git config user.name "smrmaheshkumar"
              '''

              // Commit and Push
              withEnv(["GIT_URL=https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME}.git"]) {
                sh '''
                  git add deployment.yml
                  git diff-index --quiet HEAD || git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                  git push ${GIT_URL} HEAD:main || \
                    { echo "Push failed, retrying..."; sleep 5; git push ${GIT_URL} HEAD:main; }
                '''
              }
            }
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
