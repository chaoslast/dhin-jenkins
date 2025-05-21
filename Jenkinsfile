pipeline {
    agent any

    environment {
        DEPLOY_USER = 'user'
        DEPLOY_HOST = '10.0.50.8'
        BASE_DIR = '/var/www'
        CURRENT_LINK = "${BASE_DIR}/webapp"
    }

    stages {
        stage('Determine Target') {
            steps {
                script {
                    def result = sh(
                        script: "ssh ${DEPLOY_USER}@${DEPLOY_HOST} 'readlink ${CURRENT_LINK}' || echo none'",
                        returnStdout: true
                    ).trim()

                    if (result.contains('webapp_blue')) {
                        env.TARGET = 'webapp_green'
                    } else {
                        env.TARGET = 'webapp_blue'
                    }

                    env.TARGET_DIR = "${BASE_DIR}/${env.TARGET}"
                    echo "🎯 이번 배포 디렉토리: ${env.TARGET_DIR}"
                }
            }
        }

        stage('Deploy to Target') {
            steps {
                sshagent (credentials: ['webserver-key']) {
                    sh """
                        echo '📦 ${env.TARGET_DIR}에 배포 중...'
                        ssh ${DEPLOY_USER}@${DEPLOY_HOST} 'mkdir -p ${env.TARGET_DIR}'
                        scp index.html ${DEPLOY_USER}@${DEPLOY_HOST}:${env.TARGET_DIR}/index.html
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
                        echo '🔁 운영 심볼릭 링크를 새 디렉토리(${env.TARGET_DIR})로 전환 중...'
                        ssh ${DEPLOY_USER}@${DEPLOY_HOST} 'ln -snf ${env.TARGET_DIR} ${CURRENT_LINK}'
                    """
                }
            }
        }
    }
}
