pipeline {
    agent any

    tools {
        nodejs 'Node18'   // Upewnij się, że masz taką nazwę w Jenkins -> Global Tools -> NodeJS installations
    }

    environment {
        JUICE_URL = 'http://localhost:3000'
    }

    stages {
        stage('Install dependencies') {
            steps {
                sh 'npm install'
            }
        }

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
                    ghcr.io/zaproxy/zap2docker-stable zap.sh -daemon -port 8090 -host 0.0.0.0 -config api.disablekey=true
                '''
                sleep 10
            }
        }

        stage('Passive Scan') {
            steps {
                sh '''
                docker exec zap_passive zap-cli --port 8090 open-url ${JUICE_URL}
                sleep 10
                docker exec zap_passive zap-cli --port 8090 spider ${JUICE_URL}
                sleep 20
                docker exec zap_passive zap-cli --port 8090 report -o zap_passive_report.html -f html
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

