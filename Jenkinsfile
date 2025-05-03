pipeline {
    agent any

    tools {
        nodejs 'Node18'
    }

    environment {
        JUICE_URL = 'http://host.docker.internal:3000'
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

 stage('[ZAP] Passive Scan using Docker plugin') {
            steps {
                script {
                    docker.image('ghcr.io/zaproxy/zaproxy:stable').inside('--add-host=host.docker.internal:host-gateway --user 0') {
                        sh '''
                            ls -la
                            zap.sh -cmd -addonupdate
                            zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta -autorun passive_scan.yaml
                        '''
                    }
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: '**/zap_html_report.html', allowEmptyArchive: true
            archiveArtifacts artifacts: '**/zap_xml_report.xml', allowEmptyArchive: true
        }
    }
}	
