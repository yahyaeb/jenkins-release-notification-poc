
/*
 * Release notification eligibility
 *
 * Only Pull Requests whose SOURCE branch follows:
 * releaseX.X.X
 *
 * Examples:
 *   release1.0.0   -> true
 *   release2.5.12  -> true
 *   feature/login  -> false
 *   manual-pr-test -> false
 *
 * Ordinary branch builds are also excluded.
 */
def isReleasePR() {

    if (!env.CHANGE_ID || !env.CHANGE_BRANCH) {
        return false
    }

    return env.CHANGE_BRANCH ==~ /^release[0-9]+\.[0-9]+\.[0-9]+$/
}


/*
 * Send an immediate notification for a failed validation.
 *
 * The eligibility check is inside this function so that
 * every validation still executes regardless of branch name.
 */
def notifyFailure(String validationName) {

    // Step 1: Check whether this build is a relevant release PR.
    if (!isReleasePR()) {
        echo """
        ----------------------------------------
        SRE NOTIFICATION SUPPRESSED
        ----------------------------------------
        Failed validation : ${validationName}
        PR number         : ${env.CHANGE_ID ?: 'N/A'}
        Source branch     : ${env.CHANGE_BRANCH ?: 'N/A'}
        Reason            : Not a releaseX.X.X source-branch PR
        ----------------------------------------
        """
        return
    }

    // Step 2: Collect information for the notification.
    def releaseName = env.CHANGE_BRANCH
    def prNumber = env.CHANGE_ID
    def targetBranch = env.CHANGE_TARGET ?: 'Unknown'
    def prUrl = env.CHANGE_URL ?: 'Unavailable'
    def buildNumber = env.BUILD_NUMBER ?: 'Unknown'
    def buildUrl = env.BUILD_URL ?: 'Unavailable'

    echo """
    ----------------------------------------
    RELEASE VALIDATION FAILURE
    ----------------------------------------
    Release           : ${releaseName}
    Failed validation : ${validationName}
    PR number         : #${prNumber}
    Target branch     : ${targetBranch}
    Jenkins build     : #${buildNumber}
    Sending SRE notification...
    ----------------------------------------
    """

    // Step 3: Prepare an informative notification.
    def notificationTitle = "Release ${releaseName}: validation failed"

    def notificationBody = """RELEASE VALIDATION FAILURE

Release: ${releaseName}
Failed check: ${validationName}

Pull Request: #${prNumber}
Source: ${releaseName}
Target: ${targetBranch}

Jenkins build: #${buildNumber}

PR: ${prUrl}
Build: ${buildUrl}

Action: Investigate the failed validation in Jenkins."""

    /*
     * Use --data-binary with a temporary file so multiline
     * notification content is transmitted reliably.
     *
     * Groovy writes the message to a file instead of embedding
     * dynamic PR metadata directly into a shell command.
     */
    def notificationFile = "ntfy-message-${UUID.randomUUID().toString()}.txt"

    try {
        writeFile(
            file: notificationFile,
            text: notificationBody
        )

        def status = withEnv([
            "NTFY_TITLE=${notificationTitle}",
            "NTFY_MESSAGE_FILE=${notificationFile}"
        ]) {
            sh(
                script: '''
                    curl --fail --silent --show-error \
                        --max-time 5 \
                        --retry 0 \
                        -H "Title: ${NTFY_TITLE}" \
                        -H 'Priority: high' \
                        -H 'Tags: warning' \
                        --data-binary @"${NTFY_MESSAGE_FILE}" \
                        https://ntfy.sh/jenkins-demo-8f72a91c4e63b2
                ''',
                returnStatus: true
            )
        }

        if (status == 0) {
            echo "SRE notification sent successfully for ${releaseName} / ${validationName}"
        } else {
            echo "WARNING: ntfy notification failed (exit code: ${status})"
        }

    } finally {
    withEnv(["NTFY_MESSAGE_FILE=${notificationFile}"]) {
        sh(
            script: 'rm -f -- "$NTFY_MESSAGE_FILE"',
            returnStatus: true
        )
    }
}

pipeline {

    agent any

    stages {

        stage('Inspect PR Metadata') {

            steps {

                echo '----------------------------------------'
                echo 'BUILD / PR METADATA'
                echo '----------------------------------------'

                echo "CHANGE_ID: ${env.CHANGE_ID}"
                echo "CHANGE_AUTHOR: ${env.CHANGE_AUTHOR}"
                echo "CHANGE_BRANCH: ${env.CHANGE_BRANCH}"
                echo "CHANGE_TARGET: ${env.CHANGE_TARGET}"
                echo "CHANGE_URL: ${env.CHANGE_URL}"
                echo "BRANCH_NAME: ${env.BRANCH_NAME}"

                script {

                    if (isReleasePR()) {

                        echo """
                        ----------------------------------------
                        RELEASE PR DETECTED
                        ----------------------------------------
                        Release: ${env.CHANGE_BRANCH}

                        Failed validations WILL notify SRE.
                        All validations will execute.
                        ----------------------------------------
                        """

                    } else {

                        echo """
                        ----------------------------------------
                        NON-RELEASE BUILD DETECTED
                        ----------------------------------------
                        Source branch: ${env.CHANGE_BRANCH ?: 'N/A'}

                        SRE notifications are disabled.
                        All validations will still execute.
                        ----------------------------------------
                        """
                    }
                }
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
