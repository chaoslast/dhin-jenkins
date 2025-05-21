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

        stage('Switch Symlink to GREEN') {
            steps {
                echo "🔁 운영 링크를 GREEN으로 전환합니다"
                sshagent(['webserver-key']) {
                    sh """
                        ssh $DEPLOY_USER@$DEPLOY_HOST 'ln -snf $GREEN_DIR $SYMLINK'
                    """
                }
            }
        }
    }
}
