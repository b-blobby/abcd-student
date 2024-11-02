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
                    git credentialsId: 'github-pat', url: 'https://github.com/b-blobby/abcd-student', branch: 'main'
                }
            }
        }
        stage('Prepare') {
            steps {
                sh 'mkdir -p results/'
                
            }
        }
        stage('DAST') {
            steps {
                sh '''
                docker run --name juice-shop -d --rm \
                -p 3000:3000 bkimminich/juice-shop
                sleep 20
            '''
            sh '''
                docker run --name zap \
                    -v /home/kali/Documents/DevSecOps/abcd-student/.zap:/zap/wrk/:rw \
                    -t ghcr.io/zaproxy/zaproxy:stable \
                    bash -c "zap.sh -cmd -addonupdate; zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta -autorun /zap/wrk/passive.yml" ||true
                '''
            }
            post {
                always {
                sh '''
                    docker cp zap:/zap/wrk/reports/zap_html_report.html ${WORKSPACE}/results/zap_html_report.html
                    docker cp zap:/zap/wrk/reports/zap_xml_report.xml ${WORKSPACE}/results/zap_xml_report.xml
                    docker stop zap juice_sop
                    docker rm zap
                '''
                }
            }
        }
    }
    post {
        always {
            echo 'Archiving results...'
            archiveArtifacts artifacts: 'results/**/*', fingerprint: true, allowEmptyArchive: true
            echo 'Sending reports to defectDojo...'
            defectDojoPublisher(artifact: 'results/zap_xml_report.xml', productName: 'Juice Shop', scanType: 'Zap Scan', engagementName: 'beata.bernat96@gmail.com')
        }
    }
}
