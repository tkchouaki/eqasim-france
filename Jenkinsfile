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
                COMMIT=$(git describe --always)
                BRANCH=$(git rev-parse --abbrev-ref HEAD)
                export OUTPUT="/mnt/data/$BRANCH/$COMMIT"
                echo "Outputting to: $OUTPUT"
                mkdir -p "$OUTPUT"
                cd /mnt/data/utils
                ./prepare_config.sh "$BASE/pipeline_cache" "$BASE/pipeline_data" "$OUTPUT" "$BASE/config.yml"
                cd "$BASE"
                echo "${OUTPUT}" > output_path.txt
                '''
                script {
                    def output = sh(returnStdout: true, script: '''cat output_path.txt''').trim()
                }
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
}