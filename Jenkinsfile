pipeline {
  agent any

  parameters {
    booleanParam(name: 'SKIP_BUILD', defaultValue: false, description: 'Saltar build')
  }

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
        git branch: 'main',
            url: 'https://github.com/melifisher/springboot-app-m4.git',
            credentialsId: '1-github-token'

        sh '''
        set -ex
        echo "Repositorio descargado correctamente"
        git log -1
        '''
      }
    }

    stage('Build') {
      when {
        expression { return !params.SKIP_BUILD }
      }
      steps {
        sh '''
        set -ex
        echo "Compilando proyecto"
        mvn clean compile
        '''
      }
    }

    stage('Unit Tests & Package') {
      when {
        expression { return !params.SKIP_BUILD }
      }
      steps {
        sh '''
        set -ex
        echo "Ejecutando tests y generando JAR..."
        mvn package
        '''
      }
    }

    stage('SAST - Semgrep') {
      when {
        expression { return !params.SKIP_BUILD }
      }
      steps {
        sh '''
    set -ex

    export PATH=$PATH:/var/jenkins_home/.local/bin

    echo "Running SAST scan..."

    semgrep scan \
      --config=auto \
      --config=p/security-audit \
      --json \
      --output reports/semgrep-report.json \
      .
    '''
      }
    }

    stage('SCA - Dependency Check') {
      steps {
    sh '''
	  echo "Running SCA scan..."
  
    mvn org.owasp:dependency-check-maven:check
    '''
      }
    }

    stage('Transfer Artifact') {
      steps {
        sh '''
        set -ex

        echo "Transfiriendo artefacto al servidor..."

        scp -vvv \
          -o StrictHostKeyChecking=no \
          -i ${SSH_KEY} \
          target/${JAR_NAME} \
          ${REMOTE_USER}@${REMOTE_IP}:${REMOTE_DIR}/
        '''
      }
    }

    stage('Deploy Production (8081)') {
      steps {
        sh '''
        set -ex

        echo "Iniciando despliegue remoto..."

        ssh -vvv -i ${SSH_KEY} ${REMOTE_USER}@${REMOTE_IP} "
          echo 'Servidor:' \$(hostname)
          echo 'Directorio:' ${REMOTE_DIR}
          ls -lh ${REMOTE_DIR}

          sudo ${DEPLOY_SCRIPT} ${REMOTE_DIR}/${JAR_NAME} 8081
        "
        '''
      }
    }

    stage('DAST - OWASP ZAP') {
      steps {
        sh '''
        set -ex

        echo "Iniciando escaneo DAST con ZAP..."

        /ZAP_2.16.0/zap.sh \
          -cmd \
          -daemon \
          -port 8090 \
          -config api.disablekey=true \
          -quickurl http://${REMOTE_IP}:8081 \
          -quickout ${WORKSPACE}/zap-report.html \
          -addonupdate \
          -verbose || true
        '''
      }
    }

  }

  post {
    always {
      echo "Guardando reportes..."
      archiveArtifacts artifacts: 'reports/*.json, *.html', fingerprint: true
    }

    success {
      echo 'Despliegue completado exitosamente.'
    }

    failure {
      echo 'El pipeline falló. Revisa los logs detallados arriba.'
    }
  }
}
