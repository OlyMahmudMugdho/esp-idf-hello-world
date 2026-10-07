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
                    echo "Workspace files:"
                    ls -la

                    echo
                    echo "Checking project files..."

                    test -f CMakeLists.txt
                    test -f main/CMakeLists.txt
                    test -f main/main.c

                    echo
                    echo "Project files verified successfully."
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

                    echo
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
                            echo "Mounted project files:"
                            ls -la

                            echo
                            echo "Checking CMakeLists.txt..."
                            test -f CMakeLists.txt

                            echo
                            echo "SUCCESS: Docker mount is working."
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

                    echo
                    echo "Pulling ESP-IDF image..."

                    docker pull "${ESP_IDF_IMAGE}"

                    HOST_WORKSPACE="${JENKINS_HOST_HOME}${WORKSPACE#/var/jenkins_home}"

                    echo
                    echo "Host workspace:"
                    echo "${HOST_WORKSPACE}"

                    echo
                    echo "Starting ESP-IDF container..."

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
                            echo "========================================"
                            echo "BUILDING PROJECT"
                            echo "========================================"

                            idf.py build

                            echo
                            echo "========================================"
                            echo "BUILD COMPLETED"
                            echo "========================================"

                            echo
                            echo "Checking build artifacts..."

                            test -f build/bootloader/bootloader.bin
                            test -f build/partition_table/partition-table.bin
                            test -f build/esp-idf-hello-world.bin

                            ls -lh \
                                build/bootloader/bootloader.bin \
                                build/partition_table/partition-table.bin \
                                build/esp-idf-hello-world.bin

                            echo
                            echo "========================================"
                            echo "CREATING MERGED BINARY"
                            echo "========================================"

                            cd /project/build

                            python -m esptool --chip esp32s3 merge-bin \
                                -o /project/build/firmware-merged.bin \
                                -f raw \
                                --flash-mode dio \
                                --flash-freq 80m \
                                --flash-size 2MB \
                                0x0 bootloader/bootloader.bin \
                                0x8000 partition_table/partition-table.bin \
                                0x10000 esp-idf-hello-world.bin

                            echo
                            echo "========================================"
                            echo "MERGED BINARY CREATED"
                            echo "========================================"

                            test -f /project/build/firmware-merged.bin

                            ls -lh /project/build/firmware-merged.bin
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

                    echo
                    echo "Firmware size:"

                    du -h build/firmware-merged.bin
                '''
            }
        }
    }

    post {

        success {
            echo '========================================'
            echo 'ESP32-S3 BUILD SUCCESSFUL'
            echo '========================================'

            archiveArtifacts(
                artifacts: 'build/firmware-merged.bin',
                fingerprint: true
            )
        }

        failure {
            echo '========================================'
            echo 'ESP32-S3 BUILD FAILED'
            echo '========================================'
        }

        always {
            echo '========================================'
            echo 'ESP-IDF PIPELINE FINISHED'
            echo '========================================'
        }
    }
}
