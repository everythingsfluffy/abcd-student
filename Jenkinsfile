pipeline {
    agent any

    stages {
        stage('Start Juice Shop') {
            steps {
                sh 'nohup npm start &'
                sh 'sleep 10'
            }
        }

        stage('Check Juice Shop') {
            steps {
                sh 'curl -I http://host.docker.internal:3000 || true'
            }
        }

        stage('[ZAP] Passive Scan using Docker Plugin') {
            steps {
                script {
                    docker.image('ghcr.io/zaproxy/zaproxy:stable').inside('--add-host=host.docker.internal:host-gateway --user 0') {
                        sh '''
                            echo "====== Workspace ======"
                            pwd
			    ls -la
                            echo "====== passive_scan.yaml ======"
                            cat passive_scan.yaml || true

                            zap.sh -cmd -addonupdate
                            zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta \
                                -autorun /var/jenkins_home/workspace/JuiceTest/passive_scan.yaml

                            echo "====== Szukam reportów ======"
                            ls -la /zap/wrk/reports || true
                        '''
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                // Skopiuj artefakty jeśli powstaną
                sh '''
                    mkdir -p results
                    if [ -f reports/zap_html_report.html ]; then cp reports/zap_html_report.html results/; fi
                    if [ -f reports/zap_xml_report.xml ]; then cp reports/zap_xml_report.xml results/; fi
                '''
            }

            archiveArtifacts artifacts: 'results/*.html, results/*.xml', allowEmptyArchive: true
        }
    }
}
	
