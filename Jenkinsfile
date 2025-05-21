pipeline {
  agent any

  environment {
    WEB_SERVER = "user@10.0.50.8"
    BLUE_DIR = "/var/www/webapp_blue"
    GREEN_DIR = "/var/www/webapp_green"
    LIVE_DIR = "/var/www/webapp"
  }

  stages {
    stage('Deploy to Blue') {
      steps {
        sshagent (credentials: ['webserver-key']) {
          sh """
            echo '📦 최신 index.html을 ${BLUE_DIR}에 배포 중...'
            ssh ${WEB_SERVER} 'sudo mkdir -p ${BLUE_DIR}'
            scp index.html ${WEB_SERVER}:${BLUE_DIR}/index.html
          """
        }
      }
    }

    stage('Approval to Publish') {
      steps {
        input message: "🔍 http://[IP]/webapp_blue/index.html 에서 정상 동작을 확인했으면 '계속'을 눌러주세요"
      }
    }

    stage('Backup & Deploy to Live') {
      steps {
        sshagent (credentials: ['webserver-key']) {
          sh """
            echo '📂 기존 운영 파일을 ${GREEN_DIR}에 백업합니다...'
            ssh ${WEB_SERVER} '
              sudo mkdir -p ${GREEN_DIR}
              cp -f ${LIVE_DIR}/index.html ${GREEN_DIR}/index.html || echo "No file to backup"
            '
            echo '🚀 새로운 index.html을 운영 경로로 복사합니다...'
            ssh ${WEB_SERVER} '
              cp -f ${BLUE_DIR}/index.html ${LIVE_DIR}/index.html
            '
          """
        }
      }
    }
  }
}
