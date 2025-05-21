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
          def current = sh(
            script: "ssh $DEPLOY_USER@$DEPLOY_HOST 'readlink -f $CURRENT_LINK 2>/dev/null || echo none'",
            returnStdout: true
          ).trim()

          def target = (current == BLUE_DIR) ? "green" : "blue"
          env.TARGET_NAME = target
          env.TARGET_DIR = (target == "blue") ? BLUE_DIR : GREEN_DIR
          echo "🎯 현재 심볼릭 링크 대상: ${current}"
          echo "📌 이번 배포 대상 디렉토리: $TARGET_DIR"
        }
      }
    }

    stage('Deploy to Target') {
      steps {
        sshagent (credentials: ['webserver-key']) {
          sh """
            echo '📦 $TARGET_DIR 디렉토리에 index.html 배포 중...'
            ssh $DEPLOY_USER@$DEPLOY_HOST 'mkdir -p $TARGET_DIR && rm -f $TARGET_DIR/index.html'
            scp index.html $DEPLOY_USER@$DEPLOY_HOST:$TARGET_DIR/index.html
          """
        }
      }
    }

    stage('Approval to Switch') {
      steps {
        input message: "🔍 $TARGET_DIR 에서 정상 동작 확인 후 '계속'을 눌러주세요."
      }
    }

    stage('Switch Symbolic Link') {
      steps {
        sshagent (credentials: ['webserver-key']) {
          sh """
            echo '🔁 운영 심볼릭 링크를 $TARGET_DIR 으로 전환 중...'
            ssh $DEPLOY_USER@$DEPLOY_HOST "ln -snf $TARGET_DIR $CURRENT_LINK"
          """
        }
      }
    }
  }
}
