pipeline {
    agent any

    tools {
        nodejs 'Node18'
    }

    environment {
        JUICE_URL = 'http://localhost:3000'
    }

    stages {

        stage('Start Juice Shop') {
            steps {
                sh 'nohup npm start &'
                sh 'sleep 10'
            }
        }

        stage('Check Juice Shop') {
            steps {
                sh 'curl -I ${JUICE_URL} || true'
            }
        }

        stage('Start ZAP') {
            steps {
                sh '''
                docker run -u zap -d --name zap_passive \
                    -p 8090:8090 \
                    ghcr.io/zaproxy/zaproxy:stable \
                    zap.sh -daemon -port 8090 -host 0.0.0.0 -config api.disablekey=true
                '''
                sleep 20
            }
        }

        stage('Passive Scan with zap-baseline.py') {
            steps {
                sh '''
                docker exec zap_passive zap-baseline.py -t ${JUICE_URL} -r zap_passive_report.html || true
                docker cp zap_passive:/zap/zap_passive_report.html zap_passive_report.html
                '''
            }
        }

        stage('Archive Report') {
            steps {
                archiveArtifacts artifacts: 'zap_passive_report.html'
            }
        }
    }

    post {
        always {
            sh 'docker stop zap_passive || true'
            sh 'docker rm zap_passive || true'
        }
    }
}

