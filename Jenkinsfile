```groovy
pipeline {
    agent any

    environment {
        ESP_IDF_IMAGE = 'espressif/idf:latest'
    }

    stages {

        stage('Verify Workspace') {
            steps {
                sh '''
                    echo "Jenkins workspace:"
                    pwd

                    echo
                    echo "Workspace files:"
                    ls -la

                    test -f CMakeLists.txt
                '''
            }
        }

        stage('Build ESP32-S3 Firmware') {
            steps {
                sh '''
                    set -e

                    echo "Pulling ESP-IDF image..."
                    docker pull ${ESP_IDF_IMAGE}

                    # Jenkins runs inside a container, while the Docker
                    # daemon runs on the host. Convert the Jenkins
                    # container path to the corresponding host path.
                    HOST_WORKSPACE="${WORKSPACE#/var/jenkins_home}"
                    HOST_WORKSPACE="${JENKINS_HOST_HOME}${HOST_WORKSPACE}"

                    echo
                    echo "Jenkins workspace:"
                    echo "  ${WORKSPACE}"

                    echo "Host workspace:"
                    echo "  ${HOST_WORKSPACE}"

                    echo
                    echo "Host workspace contents:"
                    ls -la "${HOST_WORKSPACE}"

                    echo
                    echo "Starting ESP-IDF container..."

                    docker run --rm \
                        -v "${HOST_WORKSPACE}:/project" \
                        -w /project \
                        ${ESP_IDF_IMAGE} \
                        bash -c '
                            set -e

                            echo "Container workspace:"
                            pwd

                            echo
                            echo "Project files:"
                            ls -la

                            echo
                            echo "Checking CMakeLists.txt..."
                            test -f CMakeLists.txt
                            echo "CMakeLists.txt found."

                            echo
                            echo "Building ESP32-S3 firmware..."
                            idf.py build

                            echo
                            echo "Creating merged binary..."
                            idf.py merge-bin \
                                -o build/firmware-merged.bin

                            echo
                            echo "Generated firmware:"
                            ls -lh build/firmware-merged.bin
                        '
                '''
            }
        }

        stage('Verify Firmware') {
            steps {
                sh '''
                    set -e

                    echo "Checking generated firmware..."

                    test -f build/firmware-merged.bin

                    echo
                    echo "Firmware:"
                    ls -lh build/firmware-merged.bin
                '''
            }
        }
    }

    post {
        success {
            echo 'ESP32-S3 firmware build successful.'

            archiveArtifacts(
                artifacts: 'build/firmware-merged.bin',
                fingerprint: true
            )
        }

        failure {
            echo 'ESP32-S3 firmware build failed.'
        }

        always {
            echo 'ESP-IDF pipeline finished.'
        }
    }
}
```
