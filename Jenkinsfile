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
        stage('OSV') {
            steps {
                sh '''
                docker run --name juice-shop -d --rm \
                -p 3000:3000 bkimminich/juice-shop
                sleep 10
            '''
            sh '''
                osv-scanner scan --lockfile=package-lock.json > ${WORKSPACE}/results/osv-scan-results.json
                '''
            }
        }
    }
    post {
        always {
            echo 'Archiving results...'
            archiveArtifacts artifacts: 'results/**/*', fingerprint: true, allowEmptyArchive: true
            echo 'Sending reports to defectDojo...'
            defectDojoPublisher(artifact: 'results/osv-scan-results.json', productName: 'Juice Shop', scanType: 'OSV Scan', engagementName: 'beata.bernat96@gmail.com')
        }
    }
}
