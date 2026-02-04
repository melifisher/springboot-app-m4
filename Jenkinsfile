pipeline {
  agent any
  environment {
    IMAGE_NAME = "demo-ci-cd:latest"
  }
  stages {
    stage('Checkout') {
      steps {
        //checkout scm
        git branch: 'main', url: 'https://github.com/melifisher/springboot-app-m4.git', credentialsId: 'df06cc5f-87aa-44c2-a3dd-46554f2a3945'
        sh 'echo "Checkout del repositorio completado..."'
      }
    }
    stage('Build') {
      steps {
          sh 'mvn clean install'
         }
    }
    stage('Build Docker Image') {
      steps {
        //sh 'docker build -t $IMAGE_NAME .'
        sh 'echo "Building docker image..."'
      }
    }
    stage('Deploy') {
      steps {
        sh 'echo "deploying app jar..."'
        sh 'scp -i /var/jenkins_home/.ssh/id_ed25519 target/*.jar osboxes@192.168.1.161:/home/osboxes/artifacts/spring-boot-private/'
        sh 'ssh osboxes@192.168.0.30 -i /var/jenkins_home/.ssh/id_ed25519 "sudo /opt/spring-boot-app/deploy.sh /home/osboxes/artifacts/spring-boot-private/demo-0.0.1-SNAPSHOT.jar"'
        sh 'echo "Deploy completado"'
      }
    }
  }
  post {
    always {
      //junit '**/target/surefire-reports/*.xml'
      archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
    }
  }
}
