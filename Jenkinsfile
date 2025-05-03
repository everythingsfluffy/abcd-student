pipeline {
    agent any
   tools {
	nodejs 'Node18'
}


    environment {
        JUICE_PORT = "3000"
        ZAP_CONTAINER = "zap_scanner"
    }

    stages {
        stage('Install dependencies (only once)') {
            when {
                not {
                    expression { fileExists('node_modules') }
                }
            }
            steps {
                sh 'npm install'
            }
        }

        stage('Starting Juice Shop') {
            steps {
                script {
		sh 'pwd'
		sh 'ls -la'

               
                    sh "pkill -f 'npm start' || true"

              
                    sh "nohup npm start -- --host 0.0.0.0 &"
             
                    sleep 11
		sh'''
		   echo "===== IP addrs w kontenerze Jenkinsa ====="
            hostname -I || ip addr || ifconfig || true

            echo "===== Czy Juice Shop nasłuchuje na porcie 3000? ====="
            netstat -tuln | grep 3000 || ss -tuln | grep 3000 || true
		'''
            
                    sh "curl -I http://localhost:${JUICE_PORT} || true"
                }
            }
        } 

        stage('[ZAP] Passive Scan using Docker Plugin') {
            steps {
                script {
		

                    docker.image('ghcr.io/zaproxy/zaproxy:stable').inside('--user 0') {
                        sh '''
                            echo "====== Workspace ======"
                            pwd
			    ls -la
                            echo "====== passive_scan.yaml ======"
                            cat passive_scan.yaml || true
			    mkdir -p /zap/wrk/reports
                            zap.sh -cmd -addonupdate
                            zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta \
                                -autorun /var/jenkins_home/workspace/JuiceTest/passive_scan.yaml &

	
    	ZAP_PID=$!
   	 sleep 60  # daj czas na wykonanie automatycznego scanowania
	 cp /zap/wrk/reports/*.html .
        cp /zap/wrk/reports/*.xml .    	
kill $ZAP_PID || true
			    
			   

                            '''
                    }
                }

            }
        }
    }

    post {
        always {
		script {

           archiveArtifacts artifacts: '*.html, *.xml', allowEmptyArchive: true

    }            
}
}	
}
