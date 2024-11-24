pipeline {
    agent any
    options {
        skipDefaultCheckout(true)
    }
    stages {
        stage('Code Checkout from GitHub') {
            steps {
                script {
                    cleanWs() // Clean the workspace before checking out the code
                    git credentialsId: 'github-pat', url: 'https://github.com/b-blobby/abcd-student', branch: 'main'
                }
            }
        }
        stage('Run Semgrep on Main') {
            when {
                branch 'main' // Only run this stage on the main branch
            }
            steps {
                script {
                    // Run Semgrep container with mounted workspace
                    sh '''
                    docker run --name semgrep_c --rm \
                    -v ${WORKSPACE}:/sast/wrk:rw \
                    semgrep/semgrep:pro-sha-45390a1 \
                    sh -c "mkdir -p /sast/wrk && semgrep --config=p/ci --metrics=auto --json --output /sast/wrk/semgrep-report.json"
                    '''
                }
            }
        }
    }
    post {
        always {
            script {
                // Archive Semgrep results if they exist
                if (fileExists("semgrep-report.json")) {
                    echo 'Archiving Semgrep results...'
                    archiveArtifacts artifacts: 'semgrep-report.json', fingerprint: true, allowEmptyArchive: true
                } else {
                    echo "Semgrep report not found. Skipping artifact archiving."
                }
            }
        }
    }
}
