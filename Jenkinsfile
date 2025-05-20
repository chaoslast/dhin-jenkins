pipeline {
  agent any

  environment {
    DEPLOY_USER = 'jen' // Web 서버 로그인 계정
    DEPLOY_HOST = '10.0.50.8' // Web 서버 IP
    BLUE_DIR = '/var/www/webapp-blue'
    GREEN_DIR = '/var/www/webapp_green'
    CURRENT_LINK = '/var/www/webapp' // 운영 중인 링크
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
          env.TARGET_DIR = "/var/www/webapp_${target}"
          echo "이번 배포 대상 디렉토리: $TARGET_DIR"
        }
      }
    }

    stage('Deploy to Target') {
      steps {
        sshagent (credentials: ['webserver-key']) {
          sh """
          echo 'Blue/Green 대상 디렉토리에 배포 중...'
          ssh $DEPLOY_USER@$DEPLOY_HOST 'mkdir -p $TARGET_DIR'
          scp index.html $DEPLOY_USER@$DEPLOY_HOST:$TARGET_DIR/
          """
        }
      }
    }

    stage('Approval to Switch') {
      steps {
        input message: "신규 배포 디렉토리(${env.TARGET_DIR})에서 정상 동작하는지 확인 후 OK를 눌러주세요."
      }
    }

    stage('Switch Symbolic Link') {
      steps {
        sshagent (credentials: ['webserver-key']) {
          sh """
          echo '운영 심볼릭 링크를 새 디렉토리로 전환 중...'
          ssh $DEPLOY_USER@$DEPLOY_HOST '
            ln -snf $TARGET_DIR $CURRENT_LINK
          '
          """
        }
      }
    }
  }
}

