pipeline {
    parameters {
        text(
            name: "config_overrides",
            defaultValue: "config:\n  random_seed: 1234",
            description: "Parts of yaml config to override, the default has no effect as it rewrite the same random seed"
        )
        string(
            name: "cache_server",
            defaultValue: "false",
            description: "URL of the cache server to use, False not use any server"
        )
        string(
            name: "sampling_rates",
            defaultValue: "0.001",
            description: "Space-separated list of sampling rates"
        )
    }

    agent {
        docker {
            image 'ghcr.io/eqasim-org/eqasim-france:main'
            args '  -i --entrypoint='
        }
    }

    stages {
        stage('Test param') {
            steps {
                script {
                    sh '''
                    #!bin/bash
                    echo "$sampling_rates"
                    sampling_array=($sampling_rates)
                    echo "Here"
                    for i in "${sampling_array[@]}"
                    do
                        echo "Sampling rate: $i"
                    done
                    '''
                }
            }
        }
        stage('Prepare') {
            steps {
                sh '''
                BASE=$(pwd)
                # Making sure old directories are cleared
                rm -rf pipeline_data pipeline_cache
                rm -rf pipeline_output_*

                mkdir pipeline_data
                mkdir pipeline_cache

                # Download yq to modify .yml files in command line
                python3 -c "import urllib.request; urllib.request.urlretrieve('https://github.com/mikefarah/yq/releases/latest/download/yq_linux_amd64', 'yq')"
                chmod +x yq

                # Applying the overrides
                echo "$config_overrides" > overrides.yml
                uv --no-cache run scripts/override_config.py overrides.yml config.yml
                rm overrides.yml

                # setting up common cache and data path
                ./yq -i ".working_directory = \\"$BASE/pipeline_cache\\" | .config.data_path = \\"$BASE/pipeline_data\\" | .config.output_path = \\"$BASE/output_0.1pct\\" " config.yml
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
                    uv --no-cache run scripts/download.py -y --requests verify=false --requests timeout=300 --cache-server "$cache_server" config.yml
                '''
            }
        }

        stage('Run') {
            steps {
                script {
                    def samplingRates = params.sampling_rates.tokenize()
                    for (def samplingRate in samplingRates) {
                        sh '''
                            rm -rf "pipeline_output_${samplingRate}"
                            mkdir "pipeline_output_${samplingRate}"
                            uv --no-cache run -m synpp --config sampling_rate "${samplingRate}" --config output_path "pipeline_output_${samplingRate}" config.yml
                            tar -czf pipeline_output_${samplingRate}.tar.gz pipeline_output_${samplingRate}/*
                            rm -rf "pipeline_output_${samplingRate}"
                        '''
                    }
                }
            }
        }


        stage('Cleanup') {
            steps {
                sh '''
                rm -rf pipeline_data pipeline_cache
                '''
            }
        }
    }

    post {
        success {
            archiveArtifacts artifacts: 'pipeline_output_*.tar.gz', fingerprint: true
        }
    }
}