pipeline {
    agent any
    options {
        skipDefaultCheckout(true)
    }
    stages {
        stage('Code checkout from GitHub') {
            steps {
                script {
                    cleanWs() // Clean the workspace before checking out the code
                    git credentialsId: 'github-pat', url: 'https://github.com/b-blobby/abcd-student', branch: 'main'
                }
            }
        }
        stage('Prepare') {
            steps {
                script {
                    sh 'mkdir -p results/' // Create the results directory in Jenkins workspace
                }
            }
        }
        stage('Run Semgrep') {
            steps {
                script {
                    // Run Juice Shop in Docker
                    sh '''
                    docker run --name juice-shop -d --rm \
                    -p 3000:3000 bkimminich/juice-shop
                    '''

                    // Wait for Juice Shop to start
                    sleep(10)

                    // Run Semgrep container with mounted volume
                    sh '''
                    docker run --name semgrep_c \
                    -v ${WORKSPACE}:/sast/wrk:rw \
                    semgrep/semgrep:pro-sha-45390a1 \
                    sh -c "mkdir -p /sast/wrk && semgrep --config=p/ci --metrics=off --json --output /sast/wrk/semgrep-report.json"
                    sleep(20)
                    '''
                }
            }
        }
    }
    post {
        always {
            script {
                // Check if the report exists before attempting to archive
                if (fileExists("results/semgrep-report.json")) {
                    echo 'Archiving results...'
                    archiveArtifacts artifacts: 'results/**/*', fingerprint: true, allowEmptyArchive: true

                    // Send the report to DefectDojo
                    echo 'Sending reports to DefectDojo...'
                    defectDojoPublisher(
                        artifact: 'results/semgrep-report.json', 
                        productName: 'Juice Shop', 
                        scanType: 'Semgrep JSON Report', 
                        engagementName: 'beata.bernat96@gmail.com'
                    )
                } else {
                    echo "Semgrep report not found, skipping artifact archiving and publishing."
                }
            }
        }
    }
}
