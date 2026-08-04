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
                python3 -c "import urllib.request; urllib.request.urlretrieve('https://github.com/mikefarah/yq/releases/latest/download/yq_linux_amd64', 'yq')"
                chmod +x yq
                ./yq -i ".working_directory = \"$BASE/pipeline_cache\" | .config.data_path = \"$BASE/pipeline_data\" | .config.output_path = \"$OUTPUT\" " config.yml
                cat config.yml
                # cd /mnt/data/utils
                # ./prepare_config.sh "$BASE/pipeline_cache" "$BASE/pipeline_data" "$OUTPUT" "$BASE/config.yml"
                # cd "$BASE"
                '''
            }
        }

        stage('DownloadData') {
            steps {
                sh 'rm -rf .home && mkdir .home'
                sh '''
                    export HOME=$(pwd)/.home
                    uv --no-cache sync
                    export https_proxy=$download_proxy
                    uv --no-cache run scripts/download.py -y --no-check-certificate --timeout 300 config.yml
                    unset https_proxy
                '''
            }
        }

        stage('RunPipeline') {
            steps {
                sh '''
                    uv --no-cache run -m synpp config.yml
                '''
            }
        }

        stage('Cleanup') {
            steps {
                sh '''
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
            archiveArtifacts artifacts: 'output_0.1pct.tar.gz', fingerprint: true
        }
    }
}