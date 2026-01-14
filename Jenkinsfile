pipeline {
    agent any

    options {
        skipStagesAfterUnstable()
    }

    stages {
        stage('Test') {
            steps {
                echo "Phase Test: Lancement des tests unitaires"
                bat 'gradlew.bat clean test'

                echo "Archivage des résultats des tests unitaires"
                junit 'build/test-results/test/*.xml'

                echo "Génération des rapports de tests Cucumber"
                script {
                    try {
                        bat 'gradlew.bat generateCucumberReports'
                        publishHTML([
                            allowMissing: true,
                            alwaysLinkToLastBuild: true,
                            keepAll: true,
                            reportDir: 'build/reports/cucumber/html',
                            reportFiles: 'overview-features.html',
                            reportName: 'Cucumber Report'
                        ])
                    } catch (Exception e) {
                        echo "Avertissement: Impossible de générer les rapports Cucumber: ${e.message}"
                    }
                }
            }
        }

        stage('Code Analysis') {
            steps {
                echo "Analyse du code avec SonarQube"
                withSonarQubeEnv('sonar') {
                    bat 'gradlew.bat sonar'
                }
            }
        }

        stage('Code Quality') {
            steps {
                echo "Vérification des Quality Gates"
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build') {
            steps {
                echo "Génération du Jar et de la documentation"
                bat 'gradlew.bat jar javadoc'

                echo "Archivage du fichier Jar"
                archiveArtifacts artifacts: 'build/libs/*.jar', fingerprint: true

                echo "Archivage de la documentation"
                archiveArtifacts artifacts: 'build/docs/javadoc/**', fingerprint: true, allowEmptyArchive: true
            }
        }

        stage('Deploy') {
            steps {
                echo "Déploiement du Jar sur Maven repo"
                bat 'gradlew.bat publish'
            }
        }
    }

    post {
        always {
            echo "Nettoyage et archivage des artefacts"
        }

        success {
            echo "Pipeline terminé avec succès: Notification par mail et Slack"
            script {
                // Email (uses environment variables)
                try {
                    def emailUser = System.getenv('EMAIL_USER')
                    def emailTo   = System.getenv('EMAIL_TO')
                    if (emailUser && emailTo) {
                        emailext(
                            to: emailTo,
                            subject: "✅ Pipeline Success: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                            body: """
                                <h2>Pipeline exécuté avec succès</h2>
                                <p><strong>Projet:</strong> ${env.JOB_NAME}</p>
                                <p><strong>Build:</strong> #${env.BUILD_NUMBER}</p>
                                <p><strong>Statut:</strong> SUCCESS</p>
                                <p><strong>URL:</strong> ${env.BUILD_URL}</p>
                                <p>Le déploiement a été effectué avec succès.</p>
                            """,
                            mimeType: 'text/html'
                        )
                        echo "✓ Email de succès envoyé"
                    } else {
                        echo "⚠️ Email non envoyé: variables EMAIL_USER ou EMAIL_TO manquantes"
                    }
                } catch (Exception e) {
                    echo "✗ Erreur lors de l'envoi de l'email: ${e.message}"
                }

                // Slack (uses environment variable)
                try {
                    def slackWebhook = System.getenv('SLACK_WEBHOOK')
                    if (slackWebhook) {
                        slackSend(
                            channel: '#ogl',
                            color: 'good',
                            message: """
                                ✅ *Pipeline réussi*
                                *Projet:* ${env.JOB_NAME}
                                *Build:* #${env.BUILD_NUMBER}
                                *Statut:* SUCCESS
                                *Détails:* ${env.BUILD_URL}
                            """.stripIndent()
                        )
                        echo "✓ Notification Slack envoyée"
                    } else {
                        echo "⚠️ Notification Slack non envoyée: variable SLACK_WEBHOOK manquante"
                    }
                } catch (Exception e) {
                    echo "✗ Erreur lors de l'envoi Slack: ${e.message}"
                }
            }
        }

        failure {
            echo "Pipeline échoué: Notification par mail et Slack"
            script {
                def emailUser = System.getenv('EMAIL_USER')
                def emailTo   = System.getenv('EMAIL_TO')
                def slackWebhook = System.getenv('SLACK_WEBHOOK')

                if (emailUser && emailTo) {
                    try {
                        emailext(
                            to: emailTo,
                            subject: "❌ Pipeline Failure: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                            body: """
                                <h2>Pipeline échoué</h2>
                                <p><strong>Projet:</strong> ${env.JOB_NAME}</p>
                                <p><strong>Build:</strong> #${env.BUILD_NUMBER}</p>
                                <p><strong>Statut:</strong> ${currentBuild.currentResult}</p>
                                <p><strong>URL:</strong> ${env.BUILD_URL}</p>
                                <p>Veuillez consulter les logs pour plus de détails.</p>
                            """,
                            mimeType: 'text/html'
                        )
                        echo "✓ Email d'échec envoyé"
                    } catch (Exception e) {
                        echo "✗ Erreur lors de l'envoi de l'email: ${e.message}"
                    }
                } else {
                    echo "⚠️ Email d'échec non envoyé: variables manquantes"
                }

                if (slackWebhook) {
                    try {
                        slackSend(
                            channel: '#ogl',
                            color: 'danger',
                            message: """
                                ❌ *Pipeline échoué*
                                *Projet:* ${env.JOB_NAME}
                                *Build:* #${env.BUILD_NUMBER}
                                *Statut:* FAILURE
                                *Détails:* ${env.BUILD_URL}console
                                ⚠️ Consultez les logs pour plus d'informations
                            """.stripIndent()
                        )
                        echo "✓ Notification Slack envoyée"
                    } catch (Exception e) {
                        echo "✗ Erreur lors de l'envoi Slack: ${e.message}"
                    }
                } else {
                    echo "⚠️ Notification Slack d'échec non envoyée: variable SLACK_WEBHOOK manquante"
                }
            }
        }

        unstable {
            echo "Pipeline instable: Notification par mail et Slack"
            script {
                def emailUser = System.getenv('EMAIL_USER')
                def emailTo   = System.getenv('EMAIL_TO')
                def slackWebhook = System.getenv('SLACK_WEBHOOK')

                if (emailUser && emailTo) {
                    try {
                        emailext(
                            to: emailTo,
                            subject: "⚠️ Pipeline Unstable: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                            body: """
                                <h2>Pipeline instable</h2>
                                <p><strong>Projet:</strong> ${env.JOB_NAME}</p>
                                <p><strong>Build:</strong> #${env.BUILD_NUMBER}</p>
                                <p><strong>Statut:</strong> UNSTABLE</p>
                                <p><strong>URL:</strong> ${env.BUILD_URL}</p>
                            """,
                            mimeType: 'text/html'
                        )
                        echo "✓ Email d'instabilité envoyé"
                    } catch (Exception e) {
                        echo "✗ Erreur lors de l'envoi de l'email: ${e.message}"
                    }
                } else {
                    echo "⚠️ Email d'instabilité non envoyé: variables manquantes"
                }

                if (slackWebhook) {
                    try {
                        slackSend(
                            channel: '#ogl',
                            color: 'warning',
                            message: """
                                ⚠️ *Pipeline instable*
                                *Projet:* ${env.JOB_NAME}
                                *Build:* #${env.BUILD_NUMBER}
                                *Statut:* UNSTABLE
                                *Détails:* ${env.BUILD_URL}
                            """.stripIndent()
                        )
                        echo "✓ Notification Slack envoyée"
                    } catch (Exception e) {
                        echo "✗ Erreur lors de l'envoi Slack: ${e.message}"
                    }
                } else {
                    echo "⚠️ Notification Slack d'instabilité non envoyée: variable SLACK_WEBHOOK manquante"
                }
            }
        }
    }
}

