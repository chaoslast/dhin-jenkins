pipeline {
    agent any
    environment {
        DEPLOY_USER = 'user'
        DEPLOY_HOST = '10.0.50.8'
        TEST_DIR = '/var/www/webapp_blue'
        LIVE_DIR = '/var/www/webapp_green'
        CURRENT_LINK = '/var/www/webapp'
    }
    stages {
        stage('Prepare Test Directory') {
            steps {
                sshagent(['webserver-key']) {
                    sh """
                        ssh $DEPLOY_USER@$DEPLOY_HOST 'sudo mkdir -p $TEST_DIR'
                        ssh $DEPLOY_USER@$DEPLOY_HOST 'sudo chown user:user $TEST_DIR'
                        scp index.html $DEPLOY_USER@$DEPLOY_HOST:$TEST_DIR/index.html
                    """
                }
            }
        }

        stage('Manual Test Confirmation') {
            steps {
                input message: "🔍 테스트용 ${TEST_DIR}에서 정상 확인 후 계속 진행하려면 '계속' 클릭"
            }
        }

        stage('Deploy to Live Directory') {
            steps {
                sshagent(['webserver-key']) {
                    sh """
                        ssh $DEPLOY_USER@$DEPLOY_HOST 'sudo mkdir -p $LIVE_DIR'
                        ssh $DEPLOY_USER@$DEPLOY_HOST 'sudo chown user:user $LIVE_DIR'
                        scp index.html $DEPLOY_USER@$DEPLOY_HOST:$LIVE_DIR/index.html
                    """
                }
            }
        }

        stage('Switch Symbolic Link') {
            steps {
                sshagent(['webserver-key']) {
                    sh """
                        ssh $DEPLOY_USER@$DEPLOY_HOST 'ln -snf $LIVE_DIR $CURRENT_LINK'
                    """
                }
            }
        }
    }
}
