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
        stage('Determine Target') {
            steps {
                script {
                    def current = sh(script: "ssh $DEPLOY_USER@$DEPLOY_HOST 'readlink $SYMLINK' || echo none", returnStdout: true).trim()
                    TARGET_DIR = (current == GREEN_DIR) ? BLUE_DIR : GREEN_DIR
                    echo "🎯 이번 배포 디렉토리: ${TARGET_DIR}"
                }
            }
        }

        stage('Deploy to Target') {
            steps {
                sshagent(['webserver-key']) {
                    sh """
                        ssh $DEPLOY_USER@$DEPLOY_HOST 'mkdir -p ${TARGET_DIR}'
                        scp index.html $DEPLOY_USER@$DEPLOY_HOST:${TARGET_DIR}/index.html
                    """
                }
            }
        }

        stage('Test Confirmation') {
            steps {
                input message: "🔍 ${TARGET_DIR}에서 정상 확인 후 계속하려면 '계속' 클릭"
            }
        }

        stage('Switch Symlink') {
            steps {
                sshagent(['webserver-key']) {
                    sh """
                        ssh $DEPLOY_USER@$DEPLOY_HOST 'ln -snf ${TARGET_DIR} ${SYMLINK}'
                    """
                }
            }
        }
    }
}
