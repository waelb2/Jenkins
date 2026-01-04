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
                    // Generate Cucumber reports
                    sh './gradlew cucumberReports'
                    cucumber fileIncludePattern: '**/cucumber.json'
                }
            }
        }

        // ===========================
        stage('Code Analysis') {
            steps {
                sh './gradlew sonar'
            }
        }

        // ===========================
        stage('Code Quality') {
            steps {
                script {
                    // Check SonarQube Quality Gate
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
                    // Build JAR
                    sh './gradlew build'
                    // Generate JavaDocs
                    sh './gradlew generateDocs'
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
                // Slack notification
                slackSend(channel: '#general', color: 'good', message: "✅ Build and deployment successful: ${env.JOB_NAME} #${env.BUILD_NUMBER}")
                // Email notification
                mail to: 'lw_bouguessa@esi.dz',
                     subject: "Build Success: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                     body: "The project has been built and deployed successfully.\nCheck Jenkins for details: ${env.BUILD_URL}"
            }
        }
        failure {
            script {
                // Slack notification
                slackSend(channel: '#general', color: 'danger', message: "❌ Build failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}")
                // Email notification
                mail to: 'lw_bouguessa@esi.dz',
                     subject: "Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                     body: "The build or deployment failed.\nCheck Jenkins for details: ${env.BUILD_URL}"
            }
        }
    }
}
