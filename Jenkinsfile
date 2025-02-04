pipeline {
  agent {
    label 'node1-build'
  }
  stages {
    stage('Checkout') {
      steps {
        sh 'echo passed'
        //git branch: 'main', url: 'http://github.com/vijayarajult2/Jenkins-Zero-To-Hero.git'
      }
    }
    stage('Build and Test') {
      steps {
        sh 'ls -ltr'
        // build the project and create a JAR file
        sh './mvnw package'
      }
    }
    stage('Static Code Analysis') {
      environment {
        SONAR_URL = "http://54.89.248.50:9000"
      }
      steps {
        withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
          sh 'mvn sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL}'
        }
      }
    }
    stage('Build and Push Docker Image') {
      environment {
        DOCKER_IMAGE = "vijayarajult2/jpetstore:${BUILD_NUMBER}"
        // DOCKERFILE_LOCATION = "k8s/Dockerfile"
        REGISTRY_CREDENTIALS = credentials('docker-cred')
      }
      steps {
        script {
            sh 'docker build -t ${DOCKER_IMAGE} .'
            def dockerImage = docker.image("${DOCKER_IMAGE}")
            docker.withRegistry('https://index.docker.io/v1/', "docker-cred") {
                dockerImage.push()
            }
        }
      }
    }
    stage('Update Deployment File') {
        environment {
            GIT_REPO_NAME = "jpetstore-6-2"
            GIT_USER_NAME = "vijayrajuyj1"
        }
        steps {
            withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                sh '''
                    git config user.email "vijayarajuyj1@gmail.com"
                    git config user.name "vijayrajuyj1"
                    BUILD_NUMBER=${BUILD_NUMBER}
                    sed -i "s/{{ .Values.image.tag }}/${BUILD_NUMBER}/g" k8s/deployment.yaml
                    git add k8s/deployment.yaml
                    git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                    git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                '''
            }
        }
    }
  }
}
