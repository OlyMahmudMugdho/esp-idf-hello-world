pipeline {
    agent any

    environment {
        ESP_IDF_IMAGE = 'espressif/idf:latest'
        JENKINS_HOST_HOME = '/home/oly_ice_pust/jenkins_home'
    }

    stages {

        stage('Verify Workspace') {
            steps {
                sh '''
                    set -e

                    echo "========================================"
                    echo "VERIFY JENKINS WORKSPACE"
                    echo "========================================"

                    echo "Workspace:"
                    pwd

                    echo
                    echo "Files:"
                    ls -la

                    echo
                    echo "Checking project..."
                    test -f CMakeLists.txt
                    test -f main/CMakeLists.txt
                    test -f main/main.c

                    echo
                    echo "Project verified."
                '''
            }
        }

        stage('Test Docker Mount') {
            steps {
                sh '''
                    set -e

                    echo "========================================"
                    echo "TEST DOCKER MOUNT"
                    echo "========================================"

                    HOST_WORKSPACE="${JENKINS_HOST_HOME}${WORKSPACE#/var/jenkins_home}"

                    echo "Jenkins workspace:"
                    echo "${WORKSPACE}"

                    echo
                    echo "Host workspace:"
                    echo "${HOST_WORKSPACE}"

                    echo
                    echo "Testing Docker mount..."

                    docker run --rm \
                        -v "${HOST_WORKSPACE}:/project" \
                        -w /project \
                        ${ESP_IDF_IMAGE} \
                        bash -c '
                            set -e

                            echo "Container workspace:"
                            pwd

                            echo
                            echo "Mounted files:"
                            ls -la

                            echo
                            echo "Checking CMakeLists.txt..."
                            test -f CMakeLists.txt

                            echo
                            echo "SUCCESS: Docker mount works."
                        '
                '''
            }
        }

        stage('Build ESP32-S3 Firmware') {
            steps {
                sh '''
                    set -e

                    echo "========================================"
                    echo "BUILD ESP32-S3 FIRMWARE"
                    echo "========================================"

                    echo "Pulling ESP-IDF image..."
                    docker pull "${ESP_IDF_IMAGE}"

                    HOST_WORKSPACE="${JENKINS_HOST_HOME}${WORKSPACE#/var/jenkins_home}"

                    echo
                    echo "Host workspace:"
                    echo "${HOST_WORKSPACE}"

                    docker run --rm \
                        -v "${HOST_WORKSPACE}:/project" \
                        -w /project \
                        ${ESP_IDF_IMAGE} \
                        bash -c '
                            set -e

                            echo "========================================"
                            echo "ESP-IDF CONTAINER"
                            echo "========================================"

                            echo
                            echo "ESP-IDF version:"
                            idf.py --version

                            echo
                            echo "Working directory:"
                            pwd

                            echo
                            echo "Project files:"
                            ls -la

                            echo
                            echo "Checking CMakeLists.txt..."
                            test -f CMakeLists.txt

                            echo
                            echo "Building..."
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

                    echo "========================================"
                    echo "VERIFY FIRMWARE"
                    echo "========================================"

                    test -f build/firmware-merged.bin

                    echo
                    echo "Firmware generated successfully:"
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
