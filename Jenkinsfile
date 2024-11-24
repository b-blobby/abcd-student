pipeline {
    agent any
    options {
        skipDefaultCheckout(true)
    }
    stages {
        stage('Code checkout from GitHub') {
            steps {
                script {
                    cleanWs()  // Clean the workspace before checking out the code
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
                    docker run --name semgrep --rm -d \
                    -v //c/Users/user/Documents/ABCD/abcd-student/sast:/sast/wrk:rw \
                    returntocorp/semgrep:latest semgrep --config p/ci --json > /sast/wrk/semgrep-report.json
                    '''
                    
                    // Wait for Semgrep to finish (sleep time can be adjusted as needed)
                    sleep(20)
                }
            }
        }
    }
    post {
        always {
            script {
                // Copy Semgrep report from the running container to the Jenkins workspace
                sh '''
                docker cp semgrep:/sast/wrk/semgrep-report.json ${WORKSPACE}/results/semgrep-report.json
                '''
                
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
