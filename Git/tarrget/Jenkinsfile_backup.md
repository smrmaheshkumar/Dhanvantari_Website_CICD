pipeline {
  agent {
    docker {
      image 'smrmaheshkumar/maven-abhishek-docker-agent:v1'
      args '--user root -v /var/run/docker.sock:/var/run/docker.sock'
    }
  }

  environment {
    DOCKER_IMAGE = "smrmaheshkumar/dhanvantari-cicd:${BUILD_NUMBER}"
    NEXUS_REGISTRY = "52.43.124.187:5000"
    NEXUS_IMAGE = "${NEXUS_REGISTRY}/dhanvantari-cicd:${BUILD_NUMBER}"
    SONAR_URL = "http://35.89.216.127:9000"
    GIT_REPO_NAME = "Dhanvantari_Website_CICD"
    GIT_USER_NAME = "smrmaheshkumar"
    DOCKER_REGISTRY = "index.docker.io/v1/"
  }

  stages {
    stage('Checkout SCM') {
      steps {
        checkout([
          $class: 'GitSCM',
          branches: [[name: 'prod']],
          extensions: [
            [$class: 'CleanBeforeCheckout'],
            [$class: 'RelativeTargetDirectory', relativeTargetDir: '.']
          ],
          userRemoteConfigs: [[
            url: "https://github.com/${env.GIT_USER_NAME}/${env.GIT_REPO_NAME}.git"
          ]]
        ])
      }
    }

    stage('Build and Test') {
      steps {        
        sh 'mvn clean install package'
        archiveArtifacts artifacts: 'target/*.war', fingerprint: true
      }
    }

    stage('Static Code Analysis') {
      steps {
        withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
          sh 'mvn sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL}'
        }
      }
    }

    stage('Build Docker Image') {
      steps {
        dir('webapp/target') {
          sh "docker build --pull --no-cache -t ${env.DOCKER_IMAGE} ."
        }
      }
    }

    stage('Push to Docker Hub') {
      steps {
        script {
          docker.withRegistry("https://${env.DOCKER_REGISTRY}", 'docker-cred') {
            docker.image("${env.DOCKER_IMAGE}").push()
          }
        }
      }
    }

    stage('Push to Nexus') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'nexus-docker-cred',
          usernameVariable: 'NEXUS_USER',
          passwordVariable: 'NEXUS_PASS'
        )]) {
          script {
            try {
              sh """
                docker tag ${env.DOCKER_IMAGE} ${env.NEXUS_IMAGE}
                docker --config /tmp/.docker login ${env.NEXUS_REGISTRY} \
                  -u ${NEXUS_USER} -p ${NEXUS_PASS}
                docker --config /tmp/.docker push ${env.NEXUS_IMAGE}
              """
            } finally {
              sh 'rm -rf /tmp/.docker || true'
              sh "docker logout ${env.NEXUS_REGISTRY} || true"
            }
          }
        }
      }
    }

    stage('Update Deployment') {
      steps {
        withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
          script {
            dir('kubernetes/') {
              sh '''
                git reset --hard HEAD
                git clean -fdx
                git pull origin main
              '''
              sh "sed -i 's/replaceImageTag/${env.BUILD_NUMBER}/g' deployment.yml"
              sh '''
                git config user.email "smr.maheshkumar@gmail.com"
                git config user.name "smrmaheshkumar"
              '''
              withEnv(["GIT_URL=https://${GITHUB_TOKEN}@github.com/${env.GIT_USER_NAME}/${env.GIT_REPO_NAME}.git"]) {
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
}
