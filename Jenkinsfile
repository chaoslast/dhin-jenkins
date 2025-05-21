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
            ssh ${WEB_SERVER} 'sudo chown user:user ${BLUE_DIR}'
            scp index.html ${WEB_SERVER}:${BLUE_DIR}/index.html
          """
        }
      }
    }

    stage('Approval to Publish') {
      steps {
        input message: "🔍 http://[IP]/webapp_blue/index.html 에서 정상 동작 확인 후 계속하세요."
      }
    }

    stage('Backup & Deploy to Live') {
      steps {
        sshagent (credentials: ['webserver-key']) {
          sh """
            echo '📂 현재 운영 index.html을 백업 (green) 중...'
            ssh ${WEB_SERVER} '
              sudo mkdir -p ${GREEN_DIR}
              sudo chown user:user ${GREEN_DIR}
              if [ -f ${LIVE_DIR}/index.html ]; then
                cp -f ${LIVE_DIR}/index.html ${GREEN_DIR}/index.html
              else
                echo "기존 운영 파일 없음, 백업 생략"
              fi
            '

            echo '🚀 blue에서 운영 index.html로 반영 중...'
            ssh ${WEB_SERVER} '
              cp -f ${BLUE_DIR}/index.html ${LIVE_DIR}/index.html
            '
          """
        }
      }
    }

    stage('Rollback Option') {
      steps {
        script {
          def doRollback = input(
            id: 'userInput', message: '⚠️ 롤백할까요?', parameters: [
              booleanParam(defaultValue: false, description: '기존 운영 상태로 롤백합니다.', name: 'Rollback')
            ]
          )

          if (doRollback) {
            echo '⏪ 롤백을 진행합니다...'
            sshagent (credentials: ['webserver-key']) {
              sh """
                ssh ${WEB_SERVER} '
                  if [ -f ${GREEN_DIR}/index.html ]; then
                    cp -f ${GREEN_DIR}/index.html ${LIVE_DIR}/index.html
                  else
                    echo "⚠️ 롤백 파일이 없습니다. 롤백 불가"
                    exit 1
                  fi
                '
              """
            }
          } else {
            echo '✅ 롤백 없이 완료됩니다.'
          }
        }
      }
    }
    stage('Clean Up Blue') {
      steps {
        sshagent (credentials: ['webserver-key']) {
          sh """
            echo '🧹 blue 디렉터리를 삭제 중...'
            ssh ${WEB_SERVER} 'sudo rm -rf ${BLUE_DIR}'
          """
        }
      }
    }
  }
}
