// Lab 08 — CI Pipeline Jenkinsfile
// Deliverable written as required by the lab task sheet.
// STATUS: BLOCKED — a running Jenkins instance with a Multibranch Pipeline job and
// a GitHub webhook is required to execute and demonstrate automatic triggering.
// The Jenkinsfile is syntactically complete and follows the lab spec.
// Added stages for testing the database

pipeline {
    agent any
    tools {
        jdk 'JDK21'
    }
    environment {
        IMAGE_NAME = "team-skeleton"
    }

    stages {

        stage('Checkout') {
            steps {
                // Checks out the branch that triggered this build.
                // In a Multibranch Pipeline this covers main, feature branches,
                // and PRs automatically — no per-branch configuration needed.
                checkout scm
            }
        }

        stage('Build Image') {
            steps {
                // Builds the Docker image using the multi-stage Dockerfile from Lab 06.
                // Tags with the Jenkins build number so every build produces a uniquely
                // tagged image — avoids overwriting previous builds' artefacts.
                dir('starter') {
                  sh "docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} ."
                }
            }
        }

        stage('Start Database') {
            steps {
               sh 'docker compose up -d postgres'
            }
        }

        stage('Test Database') {
            steps {
                sh '''
                docker compose exec -T postgres \
                  psql -U postgres -d paywealth \
                  -c "SELECT * FROM clients;"
                '''
            }
        }

        stage('Stop Containers') {
            steps {
                sh 'docker compose down'
            }
        }

        stage('Test') {
            steps {
                // Runs the Maven test suite inside the build environment.
                // -B (batch mode) suppresses interactive prompts so output is clean in logs.
                dir('starter') {
                  sh "mvn -B test"
                }
            }
            post {
                always {
                    // Publishes JUnit XML results to Jenkins regardless of pass/fail.
                    // This gives a test-trend chart in the Jenkins UI and lets branch
                    // protection rules check the test result as a status check.
                    junit allowEmptyResults: true, testResults: '**/target/surefire-reports/*.xml'
                }
            }
        }
    }

    post {
        failure {
            // Notifies the team on build failure. Replace with your notification
            // mechanism (Slack, email, Teams) once Jenkins is configured.
            echo "Build ${BUILD_NUMBER} failed — check console output."
        }
        success {
            echo "Build ${BUILD_NUMBER} passed."
        }
    }
}
