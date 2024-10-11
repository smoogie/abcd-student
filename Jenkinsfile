pipeline {
    agent any
    options {
        skipDefaultCheckout(true)
    }
    stages {
        stage('Code checkout from GitHub') {
            steps {
                script {
                    cleanWs()
                    git credentialsId: 'github-pat', url: 'https://github.com/smoogie/abcd-student.git', branch: 'dast-test'
                }
            }
        }
        stage('preset') {
            steps {
                sh 'mkdir -p results/'
            }
        }
        stage('[ZAP] Baseline passive-scan') {
            steps {
                sh '''
                    docker run --name juice-shop -d \
                        -p 3000:3000 \
                        bkimminich/juice-shop
                    sleep 5
                '''
                // --mount "src=abcd-lab,dst=/zap/wrk,volume-subpath=workspace/abc_devsecops/.zap" \
                // -v /home/lupa/repos/github/smoogie/abcd-student/.zap:/zap/wrk/:rw \
                sh '''
                    docker run --name zap \
                        --add-host=host.docker.internal:host-gateway \
                        -v /home/lupa/repos/github/smoogie/abcd-student/.zap:/zap/wrk/:rw \
                        -t ghcr.io/zaproxy/zaproxy:stable bash -c \
                        "zap.sh -cmd -addonupdate; zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta -autorun /zap/wrk/passive.yaml" || true \
                '''
            }
            post {
                always {
                    sh '''
                        docker cp zap:/zap/wrk/reports/zap_xml_report.xml ${WORKSPACE}/results/zap_xml_report.xml
                        docker stop zap juice-shop
                        docker rm zap juice-shop
                    '''
                }
            }
        }
    }
    post {
        always {
            echo "archive results"
            archiveArtifacts artifacts: 'results/**/*', fingerprint: true, allowEmptyArchive: true
            // echo "sendind to DefectDojo"
            // defectDojoPublisher(artifact: 'results/zap_xml_report.xml', productName:'Juice Shop', scanType:'ZAP Scan', engagementName:'	lukasz.pawlowski.inf@gmail.com')
        }
    }
}
