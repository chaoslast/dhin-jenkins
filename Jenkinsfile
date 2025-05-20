pipeline {
  agent any

  environment {
    DEPLOY_USER = 'user'
    DEPLOY_HOST = '10.0.50.8'
    BLUE_DIR = '/var/www/webapp_blue'
    GREEN_DIR = '/var/www/webapp_green'
    CURRENT_LINK = '/var/www/webapp'
  }

  stages {
    stage('Clone') {
      steps {
        git branch: 'blue-green', url: 'https://github.com/chaoslast/dhin-jenkins.git', credentialsId: 'git-test'
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
          echo "🎯 이번 배포 디렉토리: ${env.TARGET_DIR}"
        }
      }
    }

    stage('Deploy to Target') {
      steps {
        sshagent (credentials: ['webserver-key']) {
          sh """
          echo '📦 ${TARGET_DIR}에 배포 중...'
          ssh $DEPLOY_USER@$DEPLOY_HOST 'mkdir -p ${TARGET_DIR}'
          scp index.html $DEPLOY_USER@$DEPLOY_HOST:${TARGET_DIR}/index.html
          """
        }
      }
    }

    stage('Approval to Switch') {
      steps {
        input message: "🔍 ${TARGET_DIR}에서 정상 동작 확인 후 전환하려면 '계속'을 눌러주세요."
      }
    }

    stage('Switch Symbolic Link') {
      steps {
        sshagent (credentials: ['webserver-key']) {
          script {
            // Groovy 변수로 문자열 생성
            def switchCommand = "ln -snf ${env.TARGET_DIR} ${env.CURRENT_LINK}"
            sh """
            echo '🔁 운영 심볼릭 링크를 새 디렉토리로 전환 중...'
            ssh $DEPLOY_USER@$DEPLOY_HOST '${switchCommand}'
            """
          }
        }
      }
    }
  }
}
