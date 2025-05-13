pipeline {
    agent { label 'test-node' }

    environment {
        TEST_MAPPING_FILE = 'test-mapping.json'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Get Changed Files') {
            steps {
                script {
                    def output = sh(script: "git diff --name-only origin/main...HEAD", returnStdout: true)
                    changedFiles = output.trim().split("\n")
                    echo "Changed files: ${changedFiles}"
                }
            }
        }

        stage('Determine Impacted Tests') {
            steps {
                script {
                    def testMapping = readJSON file: "${TEST_MAPPING_FILE}"
                    def impactedTests = []

                    changedFiles.each { file ->
                        def tests = testMapping[file]
                        if (tests) {
                            impactedTests += tests
                        }
                    }

                    impactedTests = impactedTests.unique()
                    if (impactedTests.isEmpty()) {
                        echo "No impacted tests. Skipping test stage."
                        currentBuild.result = 'SUCCESS'
                        return
                    }

                    env.IMPACTED_TESTS = impactedTests.collect { "\"${it}\"" }.join(" ")
                    echo "Impacted tests: $IMPACTED_TESTS"
                }
            }
        }

        stage('Run Impacted Tests') {
            when {
                expression { return env.IMPACTED_TESTS != null }
            }
            steps {
                sh "./gradlew test --tests $IMPACTED_TESTS"
            }
        }
    }
}
