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
        stage('Semgrep') {
            steps {
                sh '''
                docker run --name juice-shop -d --rm \
                -p 3000:3000 bkimminich/juice-shop
                sleep 10
            '''
            sh '''
                docker run --name semgrep --rm -d returntocorp/semgrep:latest \
                -v //c/Users/user/Documents/ABCD/abcd-student/sast:/sast/wrk/:rw
                '''
            sh '''
                semgrep --config p/ci --json > /sast/wrk/semgrep-report.json
                '''
            
             post {
                always {
                sh '''
                    docker cp sast:/sast/wrk/reports/semgrep-report.json ${WORKSPACE}/results//semgrep-report.json                    
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
            defectDojoPublisher(artifact: 'results/semgrep-report.json', productName: 'Juice Shop', scanType: 'Semgrep JSON Report', engagementName: 'beata.bernat96@gmail.com')
        }
    }
}
