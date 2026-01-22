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
        dir('spring-boot-app') {
          sh 'mvn clean package -DskipTests'
        }
      }
    }

    stage('SonarQube Analysis') {
      steps {
        dir('spring-boot-app') {
          withSonarQubeEnv('sonarqube') {
            sh 'mvn sonar:sonar'
          }
        }
      }
    }

    stage('Quality Gate') {
      steps {
        waitForQualityGate abortPipeline: true
      }
    }

    stage('Tests') {
      steps {
        dir('spring-boot-app') {
          sh 'mvn test'
        }
      }
    }

    stage('Docker Build & Push') {
      steps {
       withCredentials([usernamePassword(
  credentialsId: 'github-creds',
  usernameVariable: 'GIT_USER',
  passwordVariable: 'GIT_TOKEN'
)]) {
    sh '''
      git clone https://github.com/AYUSH-1406/spring-boot-manifests.git
      cd spring-boot-manifests

      git checkout main || git checkout -b main

      cd base
      sed -i s|image:.*|image: ayush1406/spring-boot-app:${IMAGE_TAG}| deployment.yaml

      git add deployment.yaml
      git commit -m "Update image to ${IMAGE_TAG}"

      git remote set-url origin https://${GIT_USER}:${GIT_TOKEN}@github.com/AYUSH-1406/spring-boot-manifests.git
      git push origin main
    '''
}

        }
      }
    }

    stage('Update GitOps Manifests') {
      steps {
     withCredentials([usernamePassword(
  credentialsId: 'github-creds',
  usernameVariable: 'GIT_USER',
  passwordVariable: 'GIT_TOKEN'
)]) {
    sh '''
      git remote set-url origin https://${GIT_USER}:${GIT_TOKEN}@github.com/AYUSH-1406/spring-boot-manifests.git
      git push origin main
    '''
}

      }
    }
  }

  post {
    success {
      echo "✅ CI + GitOps pipeline completed successfully"
    }
    failure {
      echo "❌ CI + GitOps pipeline failed"
    }
  }
}
