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
        stage('Trufflehog') {
            steps {
                sh '''
                docker run --name juice-shop -d --rm \
                -p 3000:3000 bkimminich/juice-shop
                sleep 10
            '''
            sh '''
                trufflehog git file://. --since-commit main --branch main --only-verified --fail --json > results/trufflehog.json
                '''
            }
        }
    }
    post {
        always {
            echo 'Archiving results...'
            archiveArtifacts artifacts: 'results/**/*', fingerprint: true, allowEmptyArchive: true
            echo 'Sending reports to defectDojo...'
            defectDojoPublisher(artifact: 'results/trufflehog.json', productName: 'Juice Shop', scanType: 'Trufflehog Scan', engagementName: 'beata.bernat96@gmail.com')
        }
    }
}
