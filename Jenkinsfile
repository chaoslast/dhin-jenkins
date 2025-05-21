pipeline {
    agent any

    environment {
        DEPLOY_USER = 'user'
        DEPLOY_HOST = '10.0.50.8'
        BASE_DIR = '/var/www'
        CURRENT_LINK = "${BASE_DIR}/webapp"
    }

    stages {
        stage('Clone') {
            steps {
                git branch: 'blue-green', credentialsId: 'git-test', url: 'https://github.com/chaoslast/dhin-jenkins.git'
            }
        }

        stage('Determine Target') {
            steps {
                script {
                    def currentTarget = sh(
                        script: "ssh ${DEPLOY_USER}@${DEPLOY_HOST} 'readlink ${CURRENT_LINK}' || echo none",
                        returnStdout: true
                    ).trim()

                    if (currentTarget.endsWith('webapp_blue')) {
                        TARGET_DIR = "${BASE_DIR}/webapp_green"
                    } else {
                        TARGET_DIR = "${BASE_DIR}/webapp_blue"
                    }

                    echo "🎯 이번 배포 디렉토리: ${TARGET_DIR}"
                }
            }
        }

        stage('Deploy to Target') {
            steps {
                sshagent(['user']) {
                    sh "echo '📦 ${TARGET_DIR}에 배포 중...'"
                    sh "ssh ${DEPLOY_USER}@${DEPLOY_HOST} 'mkdir -p ${TARGET_DIR}'"
                    sh "scp index.html ${DEPLOY_USER}@${DEPLOY_HOST}:${TARGET_DIR}/index.html"
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
                sshagent(['webserver-key']) {
                    sh "echo '🔁 운영 심볼릭 링크를 새 디렉토리로 전환 중...'"
                    sh "ssh ${DEPLOY_USER}@${DEPLOY_HOST} 'ln -snf ${TARGET_DIR} ${CURRENT_LINK}'"
                }
            }
        }
    }
}
