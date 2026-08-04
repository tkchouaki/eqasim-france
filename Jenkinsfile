pipeline {
    agent {
        docker {
            image 'ghcr.io/eqasim-org/eqasim-france:main'
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
                sh 'rm -rf .home && mkdir .home'
                sh '''
                    export HOME=$(pwd)/.home
                    export https_proxy_backup=$https_proxy
                    unset https_proxy
                    uv --no-cache sync
                    export https_proxy=$https_proxy_backup
                    uv --no-cache run scripts/download.py -y --no-check-certificate --timeout 300 config.yml
                '''
            }
        }

        stage('RunPipeline') {
            steps {
                sh 'uv --no-cache run -m synpp config.yml'
            }
        }

        stage('Cleanup') {
            steps {
                sh '''
                unset $https_proxy
                rm -rf pipeline_data pipeline_cache
                rm -rf output_0.1pct.tar.gz
                tar -czf output_0.1pct.tar.gz pipeline_output/*
                rm -rf pipeline_output
                '''
            }
        }
    }

    post {
        success {
            archiveArtifacts artifacts: 'output_0.1pct.zip', fingerprint: true
        }
    }
}