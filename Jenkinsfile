pipeline {
    agent {
        docker {
            image 'ghcr.io/eqasim-org/eqasim-france:main'
            args '  -i --entrypoint='
        }
    }

    stages {
        stage('Prepare') {
            steps {
                sh '''
                BASE=$(pwd)
                export OUTPUT="$BASE/pipeline_output"
                rm -rf pipeline_data
                mkdir pipeline_data
                rm -rf pipeline_cache
                mkdir pipeline_cache
                python3 -c "import urllib.request; urllib.request.urlretrieve('https://github.com/mikefarah/yq/releases/latest/download/yq_linux_amd64', 'yq')"
                chmod +x yq
                ./yq -i ".working_directory = \\"$BASE/pipeline_cache\\" | .config.data_path = \\"$BASE/pipeline_data\\" | .config.output_path = \\"$BASE/output_0.1pct\\" " config.yml
                cp config.yml config_0.1pct.yml
                cp config.yml config_1pct.yml
                cp config.yml config_10pct.yml

                ./yq -i ".config.output_path = \\"$BASE/output_1pct\\" | .config.sampling_rate = \\"0.01\\"" confg_1pct.yml
                ./yq -i ".config.output_path = \\"$BASE/output_10pct\\" | .config.sampling_rate = \\"0.1\\"" confg_10pct.yml
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
                    uv --no-cache run -m synpp config_0.1pct.yml
                    uv --no-cache run -m synpp config_1pct.yml
                    uv --no-cache run -m synpp config_10pct.yml
                '''
            }
        }

        stage('Cleanup') {
            steps {
                sh '''
                rm -rf pipeline_data pipeline_cache
                rm -rf output_*.tar.gz
                tar -czf output_0.1pct.tar.gz output_0.1pct/*
                tar -czf output_1pct.tar.gz output_1pct/*
                tar -czf output_10pct.tar.gz output_10pct/*
                rm -rf output_0.1pct output_1pct output_10pct
                '''
            }
        }
    }

    post {
        success {
            archiveArtifacts artifacts: 'output_0.1pct.tar.gz output_1pct.tar.gz output_10pct.tar.gz', fingerprint: true
        }
    }
}