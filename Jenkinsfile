pipeline {
  agent any

  environment {
    WEB_SERVER = "user@10.0.50.8"
    BLUE_DIR = "/var/www/webapp_blue"
    GREEN_DIR = "/var/www/webapp_green"
    LIVE_DIR = "/var/www/webapp"
    SLACK_CHANNEL = "#dhin-test"
    SLACK_TOKEN_CREDENTIAL_ID = "dhin-Notice" // Jenkins에 등록된 Slack 자격증명 ID
  }

  options {
    timestamps()
  }

  stages {
    stage('Deploy to Blue') {
      steps {
        script {
          slackSend(channel: SLACK_CHANNEL, message: "📦 [Deploy] Blue 디렉터리에 파일 배포 시작합니다.")
        }
        sshagent (credentials: ['webserver-key']) {
          sh """
            echo '📦 최신 index.html을 ${BLUE_DIR}에 배포 중...'
            ssh ${WEB_SERVER} 'sudo mkdir -p ${BLUE_DIR}'
            ssh ${WEB_SERVER} 'sudo chown user:user ${BLUE_DIR}'
            scp index.html ${WEB_SERVER}:${BLUE_DIR}/index.html
          """
        }
        script {
          slackSend(channel: SLACK_CHANNEL, message: "✅ [Deploy] Blue 디렉터리 배포 완료되었습니다.")
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
        script {
          slackSend(channel: SLACK_CHANNEL, message: "🚀 [Deploy] 운영 서버 백업 및 Blue → Live 반영 시작")
        }
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
        script {
          slackSend(channel: SLACK_CHANNEL, message: "✅ [Deploy] Live 반영 완료")
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
            slackSend(channel: SLACK_CHANNEL, message: "⏪ [Rollback] 롤백 시도 중...")
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
            slackSend(channel: SLACK_CHANNEL, message: "✅ [Rollback] 롤백 완료")
          } else {
            slackSend(channel: SLACK_CHANNEL, message: "🚀 [Deploy] 롤백 없이 정상 배포 완료")
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
        script {
          slackSend(channel: SLACK_CHANNEL, message: "🧹 [CleanUp] Blue 디렉터리 삭제 완료")
        }
      }
    }
  }

  post {
    success {
      slackSend(channel: SLACK_CHANNEL, message: "🎉 [SUCCESS] 전체 배포 파이프라인 완료")
    }
    failure {
      slackSend(channel: SLACK_CHANNEL, message: "❌ [FAILURE] 파이프라인 실행 중 오류 발생")
    }
    aborted {
      slackSend(channel: SLACK_CHANNEL, message: "⚠️ [ABORTED] 파이프라인 수동 중단됨")
    }
  }
}
