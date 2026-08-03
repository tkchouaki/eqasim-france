pipeline {
    agent {
        docker {
            image 'ghcr.io/astral-sh/uv:debian'
            args '-v /mnt/data/jenkins_data:/mnt/data -i --entrypoint='
        }
    }

    stages {
        stage('Prepare') {
            steps {
                sh '''
                BASE=$(pwd)
                export OUTPUT="$BASE/pipeline_output"
                echo "Outputting to: $OUTPUT"
                rm -rf "$OUTPUT"
                mkdir -p "$OUTPUT"
                rm -rf pipeline_data
                mkdir pipeline_data
                rm -rf pipeline_cache
                mkdir pipeline_cache
                cd /mnt/data/utils
                ./prepare_config.sh "$BASE/pipeline_cache" "$BASE/pipeline_data" "$OUTPUT" "$BASE/config.yml"
                cd "$BASE"
                '''
            }
        }

        stage('DownloadData') {
            steps {
                sh 'uv --no-cache sync'
                sh 'uv --no-cache run scripts/download.py -y --no-check-certificate --timeout 300 config.yml'
            }
        }

        stage('RunPipeline') {
            steps {
                sh 'uv --no-cache run -m synpp config.yml'
            }
        }

        stage('Cleanup') {
            steps {
                sh '''echo "Finished" '''
            }
        }
    }

    post {
        success {
            archiveArtifacts artifacts: 'pipeline_output/*', fingerprint: true
        }
    }
}