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
                    git credentialsId: 'github-pat', url: 'https://github.com/smoogie/abcd-student.git', branch: 'sca_test'
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
                        docker cp zap:/zap/wrk/zap_xml_report.xml ${WORKSPACE}/results/zap_xml_report.xml
                        docker stop zap juice-shop
                        docker rm zap juice-shop
                    '''
                }
            }
        }
        stage('[OSV-SCANNER] scan of package-lock.json') {
            steps {
                sh '''
                osv-scanner scan --lockfile package-lock.json --format json > ${WORKSPACE}/results/osv_scan.json
                '''
            }
    post {
        always {
            echo "archive results"
            archiveArtifacts artifacts: 'results/**/*', fingerprint: true, allowEmptyArchive: true
            echo "sendind to DefectDojo"
            defectDojoPublisher(artifact: 'results/zap_xml_report.xml', productName:'Juice Shop', scanType:'ZAP Scan', engagementName:'	lukasz.pawlowski.inf@gmail.com')
//             defectDojoPublisher(artifact: 'results/osv_scan.json', productName:'Juice Shop', scanType:'OSV Scan', engagementName:'	lukasz.pawlowski.inf@gmail.com')
//             defectDojoPublisher(artifact: 'results/trufflehog_report.json', productName:'Juice Shop', scanType:'Trufflehog Scan', engagementName:'	lukasz.pawlowski.inf@gmail.com')
//             defectDojoPublisher(artifact: 'results/semgrep_report.json', productName:'Juice Shop', scanType:'Semgrep JSON Report', engagementName:'	lukasz.pawlowski.inf@gmail.com')
        }
    }
}
