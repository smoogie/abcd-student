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
        stage('[ZAP] Baseline passive-scan') {
            steps {
                sh '''
                    docker run --name juice-shop -d \
                        -p 3000:3000 \
                        bkimminich/juice-shop
                    sleep 5
                '''
                sh '''
                    docker run --name zap \
                        --add-host=host.docker.internal:host-gateway \
                        --mount "src=abcd-lab,dst=/zap/wrk,volume-subpath=workspace/abc_devsecops/.zap" \
                        -t ghcr.io/zaproxy/zaproxy:stable bash -c \
                        "ls /zap/wrk -al && zap.sh -cmd -addonupdate; zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta -autorun /zap/wrk/passive.yaml" \
                '''
            }
            post {
                always {
                    sh '''
                        docker cp zap:/zap/wrk/zap_xml_report.xml ${WORKSPACE}/zap_xml_report.xml
                        docker stop zap juice-shop
                        docker rm zap juice-shop
                    '''
                }
            }
        }
    }
}
