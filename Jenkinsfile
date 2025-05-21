pipeline {
    agent any
    environment {
        DEPLOY_USER = 'user'
        DEPLOY_HOST = '10.0.50.8'
        BLUE_DIR = '/var/www/webapp_blue'
        GREEN_DIR = '/var/www/webapp_green'
        SYMLINK = '/var/www/webapp'
    }
    stages {
        stage('Deploy to BLUE (Pre-test)') {
            steps {
                echo "📦 BLUE 디렉토리에 먼저 배포합니다"
                sshagent(['webserver-key']) {
                    sh """
                        ssh $DEPLOY_USER@$DEPLOY_HOST 'sudo mkdir -p $BLUE_DIR'
                        ssh $DEPLOY_USER@$DEPLOY_HOST 'sudo chown user:user $BLUE_DIR'
                        scp index.html $DEPLOY_USER@$DEPLOY_HOST:$BLUE_DIR/index.html
                    """
                }
            }
        }

        stage('Test BLUE Before Switch') {
            steps {
                input message: "🔍 http://<서버주소>/webapp_blue/index.html 에서 정상 확인되면 계속 진행"
            }
        }

        stage('Deploy to GREEN (Live Target)') {
            steps {
                echo "📦 GREEN 디렉토리에 최종 배포합니다"
                sshagent(['webserver-key']) {
                    sh """
                        ssh $DEPLOY_USER@$DEPLOY_HOST 'sudo mkdir -p $GREEN_DIR'
                        ssh $DEPLOY_USER@$DEPLOY_HOST 'sudo chown user:user $GREEN_DIR'
                        scp index.html $DEPLOY_USER@$DEPLOY_HOST:$GREEN_DIR/index.html
                    """
                }
            }
        }

        stage('Switch Symbolic Link') {
          steps {
             sshagent (credentials: ['webserver-key']) {
              sh '''
                echo '🧹 기존 webapp 디렉토리 정리 및 심볼릭 링크 전환 중...'
        
                # 1️⃣ 기존 index.html 제거 (디렉토리 자체가 물리 디렉토리일 경우만)
                if [ ! -L /var/www/webapp ]; then
                  echo '📂 기존 webapp 디렉토리가 심볼릭 링크가 아니므로 내용 삭제'
                  rm -rf /var/www/webapp/*
                  rmdir /var/www/webapp || true
               fi

                # 2️⃣ 새로운 심볼릭 링크 설정
                ln -snf /var/www/webapp_green /var/www/webapp
              '''
            }
          }
       }
    }
}
