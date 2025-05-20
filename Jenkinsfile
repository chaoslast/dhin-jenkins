pipeline {
  agent any

  environment {
    DEPLOY_USER = 'user' // Web 서버 SSH 계정
    DEPLOY_HOST = '10.0.50.8' // Web 서버 IP
    BLUE_DIR = '/var/www/webapp_blue'
    GREEN_DIR = '/var/www/webapp_green'
    CURRENT_LINK = '/var/www/webapp' // 현재 운영 중인 심볼릭 링크
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
          def target = sh(
            script: "ssh $DEPLOY_USER@$DEPLOY_HOST 'readlink $CURRENT_LINK | grep blue && echo green || echo blue'",
            returnStdout: true
          ).trim()
          env.TARGET_NAME = target
          env.TARGET_DIR = "/var/www/webapp_${target}"
          echo "🎯 이번 배포 대상 디렉토리: ${env.TARGET_DIR}"
        }
      }
    }

    stage('Deploy to Target') {
      steps {
        sshagent (credentials: ['webserver-key']) {
          sh """
          echo '📦 배포 디렉토리 생성 및 파일 전송 중...'
          ssh $DEPLOY_USER@$DEPLOY_HOST 'mkdir -p ${TARGET_DIR}'
          scp index.html $DEPLOY_USER@$DEPLOY_HOST:${TARGET_DIR}/index.html
          """
        }
      }
    }

    stage('Approval to Switch') {
      steps {
        input message: "신규 배포 디렉토리 ${TARGET_NAME} 에서 정상 동작하는지 확인 후 OK를 눌러주세요. (현재: ${TARGET_NAME})"
      }
    }

    stage('Switch Symbolic Link') {
      steps {
        sshagent (credentials: ['webserver-key']) {
          sh """
          echo '🔁 운영 심볼릭 링크를 새 디렉토리로 전환 중...'
          ssh $DEPLOY_USER@$DEPLOY_HOST 'ln -snf ${TARGET_DIR} ${CURRENT_LINK}'
          """
        }
      }
    }
    
    stage('Confirm') {
      steps {
        echo "✅ 배포 완료! ${CURRENT_LINK} → ${TARGET_DIR} 로 전환됨"
      }
    }
  }
}
