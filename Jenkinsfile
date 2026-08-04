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
                # Making sure old directories are cleared
                rm -rf pipeline_data pipeline_cache output_0.1pct output_1pct output_10pct
                mkdir pipeline_data
                mkdir pipeline_cache
                mkdir output_0.1pct
                mkdir output_1pct
                mkdir output_10pct

                # Download yq to modify .yml files in command line
                python3 -c "import urllib.request; urllib.request.urlretrieve('https://github.com/mikefarah/yq/releases/latest/download/yq_linux_amd64', 'yq')"
                chmod +x yq

                # setting up common cache and data path
                ./yq -i ".working_directory = \\"$BASE/pipeline_cache\\" | .config.data_path = \\"$BASE/pipeline_data\\" | .config.output_path = \\"$BASE/output_0.1pct\\" " config.yml

                cp config.yml config_0.1pct.yml
                cp config.yml config_1pct.yml
                cp config.yml config_10pct.yml

                # setting up different sampling rates and output paths
                ./yq -i ".config.output_path = \\"$BASE/output_1pct\\" | .config.sampling_rate = \\"0.01\\"" confg_1pct.yml
                ./yq -i ".config.output_path = \\"$BASE/output_10pct\\" | .config.sampling_rate = \\"0.1\\"" confg_10pct.yml
                '''
            }
        }

        stage('DownloadData') {
            steps {
                // Uv downloads to home, we need to set up a location that the current user is sure to be able to write into
                sh '''
                    rm -rf .home && mkdir .home
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