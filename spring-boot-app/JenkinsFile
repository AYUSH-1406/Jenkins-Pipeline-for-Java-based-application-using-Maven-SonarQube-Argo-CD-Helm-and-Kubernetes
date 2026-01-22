pipeline {
  agent any

  environment {
    IMAGE_NAME = "ayush1406/spring-boot-app"
    IMAGE_TAG  = "${GIT_COMMIT}"
  }

  tools {
    maven 'maven-3'
    jdk 'jdk-17'
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build') {
      steps {
        sh 'mvn clean package -DskipTests'
      }
    }

    stage('SonarQube Analysis') {
      steps {
        withSonarQubeEnv('sonarqube') {
          sh 'mvn sonar:sonar'
        }
      }
    }

    stage('Quality Gate') {
      steps {
        timeout(time: 10, unit: 'MINUTES') {
        waitForQualityGate abortPipeline: true
        }
      }
    }

    stage('Tests') {
      steps {
        sh 'mvn test'
      }
    }

    stage('Docker Build & Push') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'dockerhub-creds',
          usernameVariable: 'DOCKER_USER',
          passwordVariable: 'DOCKER_PASS'
        )]) {
          sh """
            echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
            docker build -t $IMAGE_NAME:$IMAGE_TAG .
            docker push $IMAGE_NAME:$IMAGE_TAG
          """
        }
      }
    }
  }

  post {
    success {
      echo "✅ CI pipeline completed successfully"
    }
    failure {
      echo "❌ CI pipeline failed"
    }
  }
}
