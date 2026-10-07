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

        stage('Verify Workspace') {
            steps {
                sh '''
                    echo "Jenkins workspace:"
                    pwd

                    echo
                    echo "Files:"
                    ls -la

                    echo
                    echo "Project CMakeLists.txt:"
                    test -f CMakeLists.txt
                '''
            }
        }

        stage('Build ESP32-S3 Firmware') {
            steps {
                sh '''
                    docker pull ${ESP_IDF_IMAGE}

                    docker run --rm \
                        -v "$WORKSPACE:/project" \
                        -w /project \
                        ${ESP_IDF_IMAGE} \
                        bash -c '
                            pwd
                            ls -la
                            idf.py build
                            idf.py merge-bin \
                                -o build/firmware-merged.bin
                        '
                '''
            }
        }

        stage('Verify Firmware') {
            steps {
                sh '''
                    test -f build/firmware-merged.bin

                    echo "Generated firmware:"
                    ls -lh build/firmware-merged.bin
                '''
            }
        }
    }

    post {
        success {
            archiveArtifacts(
                artifacts: 'build/firmware-merged.bin',
                fingerprint: true
            )
        }

        always {
            echo 'ESP-IDF pipeline finished.'
        }
    }
}
