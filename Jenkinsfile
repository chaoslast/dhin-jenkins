pipeline {
  agent any

  stages {
    stage('Clone') {
      steps {
        git branch: 'main', url: 'https://github.com/chaoslast/dhin-jenkins.git', credentialsId: 'git-test'
      }
    }

    stage('Deploy') {
      steps {
        sshagent (credentials: ['webserver-key']) {
          sh '''
            echo "[+] 파일 전송 중..."
            scp index.html user@10.0.50.8:/tmp/index.html
            ssh user@10.0.50.8 'sudo mv /tmp/index.html /var/www/webapp/index.html && sudo chown nginx:nginx /var/www/webapp/index.html'
            echo "[+] 배포 완료"
          '''
        }
      }
    }
  }
}

