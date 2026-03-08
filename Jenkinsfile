pipeline {
  agent any
  environment {
    REMOTE_IP = "192.168.0.30"
    REMOTE_USER = "melifisher"
    SSH_KEY = "/var/jenkins_home/.ssh/id_ed25519"
    REMOTE_DIR = "/home/osboxes/artifacts/spring-boot-private"
    DEPLOY_SCRIPT = "/opt/spring-boot-app/deploy.sh"
    JAR_NAME = "demo-0.0.1-SNAPSHOT.jar"
  }
  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/melifisher/springboot-app-m4.git', credentialsId: '1-github-token'
        sh 'echo "checkout del repositorio completado"'
      }
    }
    
    stage('Build') {
      steps {
        sh 'mvn clean compile'
      }
    }

    stage('Unit Tests & Package') {
      steps {
        sh 'echo "Ejecutando tests y creando JAR..."'
        sh 'mvn package'
      }
    }

    stage('SAST - Semgrep') {
      steps {
        sh 'semgrep --config=auto .'
  //              sh 'docker run --rm -v "D:\tmp\semgrep_test":/src semgrep/semgrep semgrep --config=auto --output /src/semgrep_result.json /src'
      }
    }

    stage('Transfer Artifact') {
      steps {
        sh 'echo "Enviando JAR al servidor..."'
        // Transferimos el archivo una sola vez
        sh "scp -o StrictHostKeyChecking=no -i ${SSH_KEY} target/${JAR_NAME} ${REMOTE_USER}@${REMOTE_IP}:${REMOTE_DIR}/"
      }
    }

    stage('Deploy Canary (8082)') {
      steps {
        sh 'echo "Iniciando Canary Release en puerto 8082..."'
        // AQUI ESTABA EL ERROR: Agregamos el argumento '8082' al final
        sh "ssh -i ${SSH_KEY} ${REMOTE_USER}@${REMOTE_IP} 'sudo ${DEPLOY_SCRIPT} ${REMOTE_DIR}/${JAR_NAME} 8082'"
      }
    }

    stage('Deploy Production (8081)') {
      steps {
        sh 'echo "Promoviendo a Producción en puerto 8081..."'
        // Si el paso anterior no falló, desplegamos en 8081
        sh "ssh -i ${SSH_KEY} ${REMOTE_USER}@${REMOTE_IP} 'sudo ${DEPLOY_SCRIPT} ${REMOTE_DIR}/${JAR_NAME} 8081'"
      }
    }
  }
  
  post {
    always {
      archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
    }
    success {
      echo 'Despliegue completado exitosamente en ambos nodos.'
    }
    failure {
      echo 'El despliegue falló. Revisa los logs.'
    }
  }
}