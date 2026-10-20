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