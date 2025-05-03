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

	stage('[ZAP] Baseline passive-scan') {
    	steps {
        sh 'mkdir -p results/'
                sh '''
		docker rm -f zap || true
           	 docker run --name zap \
                --add-host=host.docker.internal:host-gateway \
                -v ${WORKSPACE}:/zap/wrk/:rw \
                -t ghcr.io/zaproxy/zaproxy:stable bash -c \
                "zap.sh -cmd -addonupdate; zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta -autorun /zap/wrk/passive_scan.yaml" \
                || true
        '''
    }
    post {
        always {
            sh '''
                docker cp zap:/zap/wrk/reports/zap_html_report.html ${WORKSPACE}/results/zap_html_report.html
                docker cp zap:/zap/wrk/reports/zap_xml_report.xml ${WORKSPACE}/results/zap_xml_report.xml
                docker stop zap 
                docker rm zap
            '''
        }
    }
}
}
}
