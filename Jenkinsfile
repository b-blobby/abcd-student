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
                sh 'mkdir -p results/'  // Create the results directory
            }
        }
        stage('Semgrep') {
            steps {
                script {
                    // Run Juice Shop in Docker
                    sh '''
                    docker run --name juice-shop -d --rm \
                    -p 3000:3000 bkimminich/juice-shop
                    '''

                    // Sleep for 10 seconds to ensure Juice Shop is fully up before running Semgrep
                    sleep(10)

                    // Run Semgrep container with mounted volume
                    sh '''
                    docker run --name semgrep_c \
                    -v ${WORKSPACE}:/sast/wrk:rw \
                    semgrep/semgrep:pro-sha-45390a1 \
                    sh -c "mkdir -p /sast/wrk && semgrep --config p/ci --json > /sast/wrk/semgrep-report.json"
                    '''

                    // Copy Semgrep report to Jenkins workspace
                    sh '''
                    mkdir -p ${WORKSPACE}/results
                    docker cp semgrep_c:/sast/wrk/semgrep-report.json ${WORKSPACE}/results/semgrep-report.json
                    docker stop semgrep_c
                    docker rm semgrep_c
                    '''
                }
            }
        }
    }
    post {
        always {
            script {
                // Archive results
                echo 'Archiving results...'
                archiveArtifacts artifacts: 'results/**/*', fingerprint: true, allowEmptyArchive: true
                
                // Send the report to DefectDojo
                echo 'Sending reports to defectDojo...'
                defectDojoPublisher(
                    artifact: 'results/semgrep-report.json', 
                    productName: 'Juice Shop', 
                    scanType: 'Semgrep JSON Report', 
                    engagementName: 'beata.bernat96@gmail.com'
                )
            }
        }
    }
}
