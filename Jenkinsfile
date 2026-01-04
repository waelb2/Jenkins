pipeline {
    agent any

    tools {
        jdk 'JDK17'
    }

    environment {
        SLACK_WEBHOOK = 'https://hooks.slack.com/services/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'
        MAVEN_REPO_URL = 'https://mymavenrepo.com/repo/cEmjfkxugPlzLxXg1A2B/'
    }

    stages {

        // ===========================
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        // ===========================
        stage('Test') {
            steps {
                script {
                    sh './gradlew clean test'
                }
            }
            post {
                always {
                    // Archive JUnit results
                    junit '**/build/test-results/test/*.xml'

                    // Generate Cucumber report
                    sh './gradlew generateCucumberReports'

                    // Publish Cucumber HTML report in Jenkins
                    cucumber buildStatus: 'UNSTABLE', fileIncludePattern: '**/build/reports/cucumber/cucumber.json', jsonReportDirectory: 'build/reports/cucumber/html'
                }
            }
        }

        // ===========================
        stage('Code Analysis') {
            steps {
                sh './gradlew sonar -Psonar.skipCompile=true'
            }
        }

        // ===========================
        stage('Code Quality') {
            steps {
                script {
                    // Wait for SonarQube Quality Gate
                    def qg = waitForQualityGate()
                    if (qg.status != 'OK') {
                        error "Pipeline stopped: Quality Gate failed!"
                    }
                }
            }
        }

        // ===========================
        stage('Build') {
            steps {
                script {
                    sh './gradlew build generateDocs'
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: 'build/libs/*.jar', fingerprint: true
                    archiveArtifacts artifacts: 'build/docs/**', fingerprint: true
                }
            }
        }

        // ===========================
        stage('Deploy') {
            steps {
                sh './gradlew publish'
            }
        }
    }

    // ===========================
    post {
        success {
            script {
                slackSend(channel: '#general', color: 'good', message: "✅ Build and deployment successful: ${env.JOB_NAME} #${env.BUILD_NUMBER}")
                // You can leave email for later once SMTP is configured
                // mail(...)
            }
        }
        failure {
            script {
                slackSend(channel: '#general', color: 'danger', message: "❌ Build failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}")
                // mail(...)
            }
        }
    }
}

