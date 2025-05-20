pipeline {
  agent any

  environment {
    DEPLOY_USER = 'user' // Web 서버 로그인 계정
    DEPLOY_HOST = '10.0.50.8' // Web 서버 IP
    BLUE_DIR = '/var/www/webapp_blue'
    GREEN_DIR = '/var/www/webapp_green'
    CURRENT_LINK = '/var/www/webapp' // 운영 중인 심볼릭 링크
  }

  stages {
    stage('Clone') {
      steps {
        git branch: 'blue-green',
            url: 'https://github.com/chaoslast/dhin-jenkins.git',
            credentialsId: 'git-test'
      }
    }

    stage('Determine Target') {
      steps {
        script {
          def currentTarget = sh(
            script: """
              ssh $DEPLOY_USER@$DEPLOY_HOST '
                if [ -L "$CURRENT_LINK" ]; then
                  readlink $CURRENT_LINK
                else
                  echo "none"
                fi
              '
            """,
            returnStdout: true
          ).trim()

          echo "🔍 현재 운영 링크 대상: ${currentTarget}"

          def nextTarget = (currentTarget == "none" || currentTarget.contains("blue")) ? "green" : "blue"
          env.TARGET_NAME = nextTarget
          env.TARGET_DIR = "/var/www/webapp_${nextTarget}"

          echo "🎯 이번 배포 디렉토리: ${env.TARGET_DIR}"
        }
      }
    }

    stage('Deploy to Target') {
      steps {
        sshagent (credentials: ['webserver-key']) {
          sh """
            echo '📦 ${env.TARGET_DIR}에 배포 중...'
            ssh $DEPLOY_USER@$DEPLOY_HOST 'mkdir -p ${env.TARGET_DIR}'
            scp index.html $DEPLOY_USER@$DEPLOY_HOST:${env.TARGET_DIR}/index.html
          """
        }
      }
    }

    stage('Approval to Switch') {
      steps {
        input message: "🔍 ${env.TARGET_DIR}에서 정상 동작 확인 후 전환하려면 '계속'을 눌러주세요."
      }
    }

    stage('Switch Symbolic Link') {
      steps {
        sshagent (credentials: ['webserver-key']) {
          sh """
            echo '🔁 운영 심볼릭 링크를 새 디렉토리로 전환 중...'
            ssh $DEPLOY_USER@$DEPLOY_HOST '
              ln -snf ${env.TARGET_DIR} ${CURRENT_LINK}
            '
          """
        }
      }
    }
  }
}
