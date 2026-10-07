pipeline {
    agent any

    environment {
        ESP_IDF_IMAGE = 'espressif/idf:latest'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Pull ESP-IDF Docker Image') {
            steps {
                sh '''
                    docker pull ${ESP_IDF_IMAGE}
                '''
            }
        }

        stage('Build Firmware') {
            steps {
                sh '''
                    docker run --rm \
                        -v "$WORKSPACE:/project" \
                        -w /project \
                        ${ESP_IDF_IMAGE} \
                        idf.py build
                '''
            }
        }

        stage('Create Merged Binary') {
            steps {
                sh '''
                    docker run --rm \
                        -v "$WORKSPACE:/project" \
                        -w /project \
                        ${ESP_IDF_IMAGE} \
                        idf.py merge-bin \
                        -o build/firmware-merged.bin
                '''
            }
        }

        stage('Verify Firmware') {
            steps {
                sh '''
                    ls -lh build/firmware-merged.bin
                    file build/firmware-merged.bin
                '''
            }
        }
    }

    post {
        success {
            archiveArtifacts artifacts: 'build/firmware-merged.bin',
                           fingerprint: true
        }

        always {
            echo 'ESP-IDF build finished.'
        }
    }
}
