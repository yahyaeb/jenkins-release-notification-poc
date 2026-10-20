
def notifyFailure(String validationName) {

    echo "Sending SRE notification for: ${validationName}"

    def status = sh(
        script: """
            curl --fail --silent --show-error \\
                --max-time 5 \\
                --retry 0 \\
                -H 'Title: Jenkins Validation Failure' \\
                -H 'Priority: high' \\
                -H 'Tags: warning' \\
                -d 'Validation: ${validationName}
Build: ${env.BUILD_NUMBER}
Jenkins: ${env.BUILD_URL}' \\
                https://ntfy.sh/jenkins-demo-8f72a91c4e63b2
        """,
        returnStatus: true
    )

    if (status != 0) {
        echo "WARNING: ntfy notification failed (exit code: ${status})"
    }
}


pipeline {
    agent any

    stages {
        stage('Inspect PR Metadata') {
            steps {
                echo "CHANGE_ID: ${env.CHANGE_ID}"
                echo "CHANGE_AUTHOR: ${env.CHANGE_AUTHOR}"
                echo "CHANGE_TARGET: ${env.CHANGE_TARGET}"
                echo "CHANGE_URL: ${env.CHANGE_URL}"
                echo "BRANCH_NAME: ${env.BRANCH_NAME}"
            }
        }
        stage('Parallel Validations') {
            failFast false

            parallel {

                stage('Unit Tests') {
                    steps {
                        echo 'Starting Unit Tests'
                        sleep time: 15, unit: 'SECONDS'
                        echo 'Unit Tests completed'
                    }

                    post {
                        failure {
                            script {
                                notifyFailure('Unit Tests')
                            }
                        }
                    }
                }

                stage('Unit Tests failure 2') {
                    steps {
                        echo 'Starting Unit Tests failure 2'
                        sleep time: 15, unit: 'SECONDS'

                        echo 'ERROR: Synthetic unit test failure'

                        error('Unit Tests validation failed')
                    }

                    post {
                        failure {
                            script {
                                notifyFailure('Unit Tests failure 2')
                            }
                        }
                    }
                }

                stage('Integration Tests') {
                    steps {
                        echo 'Starting Integration Tests'
                        sleep time: 45, unit: 'SECONDS'
                        echo 'Integration Tests completed'
                    }

                    post {
                        failure {
                            script {
                                notifyFailure('Integration Tests')
                            }
                        }
                    }
                }

                stage('Security Checks') {
                    steps {
                        echo 'Starting Security Checks'
                        sleep time: 30, unit: 'SECONDS'
                        echo 'Security Checks completed'
                    }

                    post {
                        failure {
                            script {
                                notifyFailure('Security Checks')
                            }
                        }
                    }
                }

            }
        }
    }
}
