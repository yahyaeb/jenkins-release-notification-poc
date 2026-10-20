pipeline {
    agent any

    stages {
        stage('Parallel Validations') {
            parallel {
                stage('Unit Tests') {
                    steps {
                        echo 'Starting Unit Tests'
                        sleep time: 15, unit: 'SECONDS'
                        echo 'Unit Tests completed'
                    }
                }

                stage('Unit Tests failure 2') {
                    steps {
                        echo 'Starting Unit Tests'
                        sleep time: 15, unit: 'SECONDS'

                        echo 'ERROR: Synthetic unit test failure'

                        error('Unit Tests validation failed')
                    }
                    post {
    failure {
        echo 'Sending immediate SRE notification...'

        script {
            def status = sh(
                            script: '''
                                curl --fail --silent --show-error \
                                    --max-time 5 \
                                    --retry 0 \
                                    -H "Title: Jenkins Validation Failure" \
                                    -H "Priority: high" \
                                    -H "Tags: warning" \
                                    -d "Validation: Unit Tests failure 2
            Build: ${BUILD_NUMBER}
            Jenkins: ${BUILD_URL}" \
                                    https://ntfy.sh/jenkins-demo-8f72a91c4e63b2
                            ''',
                            returnStatus: true
                        )

                        if (status != 0) {
                            echo "WARNING: ntfy notification failed (exit code: ${status})"
                        }
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
                }

                stage('Security Checks') {
                    steps {
                        echo 'Starting Security Checks'
                        sleep time: 30, unit: 'SECONDS'
                        echo 'Security Checks completed'
                    }
                }
            }
        }
    }
}