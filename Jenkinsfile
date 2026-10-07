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

                    echo
                    echo "Jenkins workspace:"
                    pwd

                    echo
                    echo "Workspace path:"
                    echo "${WORKSPACE}"

                    echo
                    echo "Workspace files:"
                    ls -la

                    echo
                    echo "Checking project files..."

                    test -f CMakeLists.txt
                    test -d main
                    test -f main/CMakeLists.txt
                    test -f main/main.c

                    echo "Project files found successfully."
                '''
            }
        }

        stage('Resolve Host Workspace') {
            steps {
                sh '''
                    set -e

                    echo "========================================"
                    echo "RESOLVE HOST WORKSPACE"
                    echo "========================================"

                    HOST_WORKSPACE="${WORKSPACE#/var/jenkins_home}"
                    HOST_WORKSPACE="${JENKINS_HOST_HOME}${HOST_WORKSPACE}"

                    echo
                    echo "Jenkins workspace:"
                    echo "${WORKSPACE}"

                    echo
                    echo "Jenkins host home:"
                    echo "${JENKINS_HOST_HOME}"

                    echo
                    echo "Resolved host workspace:"
                    echo "${HOST_WORKSPACE}"

                    echo
                    echo "Host workspace contents:"
                    ls -la "${HOST_WORKSPACE}"

                    echo
                    echo "Checking host CMakeLists.txt..."
                    test -f "${HOST_WORKSPACE}/CMakeLists.txt"

                    echo "Host workspace verified successfully."
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

                    HOST_WORKSPACE="${WORKSPACE#/var/jenkins_home}"
                    HOST_WORKSPACE="${JENKINS_HOST_HOME}${HOST_WORKSPACE}"

                    echo
                    echo "Using host workspace:"
                    echo "${HOST_WORKSPACE}"

                    echo
                    echo "Starting test container..."

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
                    echo "ESP-IDF image:"
                    echo "${ESP_IDF_IMAGE}"

                    echo
                    echo "Pulling latest ESP-IDF image..."
                    docker pull "${ESP_IDF_IMAGE}"

                    HOST_WORKSPACE="${WORKSPACE#/var/jenkins_home}"
                    HOST_WORKSPACE="${JENKINS_HOST_HOME}${HOST_WORKSPACE}"

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
                            echo "Checking CMakeLists.txt..."
                            test -f CMakeLists.txt

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
                            echo "Build directory:"
                            ls -lh build

                            echo
                            echo "========================================"
                            echo "CREATING MERGED BINARY"
                            echo "========================================"

                            idf.py merge-bin \
                                -o build/firmware-merged.bin

                            echo
                            echo "Merged firmware:"
                            ls -lh build/firmware-merged.bin

                            echo
                            echo "========================================"
                            echo "ESP32-S3 FIRMWARE READY"
                            echo "========================================"
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

                    echo
                    echo "Build directory:"
                    ls -lh build

                    echo
                    echo "Checking merged firmware..."

                    test -f build/firmware-merged.bin

                    echo
                    echo "Firmware successfully generated:"
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
            echo 'JENKINS PIPELINE FINISHED'
            echo '========================================'
        }
    }
}
