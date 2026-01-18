pipeline {
    agent any

    options {
        skipStagesAfterUnstable()
        timestamps()
    }

    stages {

        stage('Checkout') {
            steps {
                echo "Récupération du code source"
                checkout scm
                sh 'chmod +x gradlew'
            }
        }

        stage('Test') {
            steps {
                echo "Phase Test : Lancement des tests unitaires"
                sh './gradlew clean test'
                echo "Archivage des résultats JUnit"
                junit allowEmptyResults: true, testResults: 'build/test-results/test/*.xml'

            }
        }

   stage('Generate HTML report') {
               steps {
                   cucumber(

                       fileIncludePattern: 'build/reports/cucumber/json-report.json'
                   )
               }
           }
        stage('Code Analysis') {
            steps {
                echo "Analyse du code avec SonarQube"
                withSonarQubeEnv('sonar') {
                    sh './gradlew sonar'
                }
            }
        }

        stage('Code Quality') {
            steps {
                echo "Vérification du Quality Gate"
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build') {
            steps {
                echo "Génération du JAR et Javadoc"
                sh './gradlew jar javadoc'

                echo "Archivage du JAR"
                archiveArtifacts artifacts: 'build/libs/*.jar', fingerprint: true

                echo "Archivage de la documentation"
                archiveArtifacts artifacts: 'build/docs/javadoc/**',
                                 fingerprint: true,
                                 allowEmptyArchive: true
            }
        }

        stage('Deploy') {
            steps {
                echo "Déploiement vers le repository Maven"
                sh './gradlew publish'
            }
        }
    }

   post {

       success {
           echo "Pipeline terminé avec succès"

           script {
               try {
                   emailext(
                       to: "mo_nemamcha@esi.dz",
                       subject: "Pipeline SUCCESS : ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                       body: """
                           <h2>Pipeline exécuté avec succès</h2>
                           <p><strong>Projet :</strong> ${env.JOB_NAME}</p>
                           <p><strong>Build :</strong> #${env.BUILD_NUMBER}</p>
                           <p><strong>Status :</strong> SUCCESS</p>
                           <p><a href="${env.BUILD_URL}">Voir le build</a></p>
                       """,
                       mimeType: 'text/html'
                   )
               } catch (e) { echo "Erreur email : ${e.message}" }

               // SLACK via curl
               withCredentials([string(credentialsId: 'SLACK_WEBHOOK', variable: 'SLACK_WEBHOOK_URL')]) {
                   sh """
                       curl -X POST -H 'Content-type: application/json' \\
                       --data '{
                           "text": "*Pipeline réussi*\\n*Projet:* ${env.JOB_NAME}\\n*Build:* #${env.BUILD_NUMBER}\\n*URL:* ${env.BUILD_URL}",
                           "username": "Jenkins",
                           "icon_emoji": ":white_check_mark:"
                       }' "$SLACK_WEBHOOK_URL"
                   """
               }
           }
       }

       failure {
           echo "Pipeline échoué"

           script {
               // EMAIL
               try {
                   emailext(
                       to: "mo_nemamcha@esi.dz",
                       subject: "Pipeline FAILURE : ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                       body: """
                           <h2>Pipeline échoué</h2>
                           <p><strong>Projet :</strong> ${env.JOB_NAME}</p>
                           <p><strong>Build :</strong> #${env.BUILD_NUMBER}</p>
                           <p><strong>Status :</strong> FAILURE</p>
                           <p><a href="${env.BUILD_URL}console">Voir les logs</a></p>
                       """,
                       mimeType: 'text/html'
                   )
               } catch (e) { echo "Erreur email : ${e.message}" }

               // SLACK via curl
               withCredentials([string(credentialsId: 'SLACK_WEBHOOK', variable: 'SLACK_WEBHOOK_URL')]) {
                   sh """
                       curl -X POST -H 'Content-type: application/json' \\
                       --data '{
                           "text": "*Pipeline échoué*\\n*Projet:* ${env.JOB_NAME}\\n*Build:* #${env.BUILD_NUMBER}\\n*Logs:* ${env.BUILD_URL}console",
                           "username": "Jenkins",
                           "icon_emoji": ":x:"
                       }' "$SLACK_WEBHOOK_URL"
                   """
               }
           }
       }

       unstable {
           echo "Pipeline instable"

           script {
               // EMAIL
               try {
                   emailext(
                       to: "oussamanemamcha@gmail.com",
                       subject: "Pipeline UNSTABLE : ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                       body: """
                           <h2>Pipeline instable</h2>
                           <p><strong>Projet :</strong> ${env.JOB_NAME}</p>
                           <p><strong>Build :</strong> #${env.BUILD_NUMBER}</p>
                           <p><strong>Status :</strong> UNSTABLE</p>
                           <p><a href="${env.BUILD_URL}">Voir le build</a></p>
                       """,
                       mimeType: 'text/html'
                   )
               } catch (e) { echo "Erreur email : ${e.message}" }

               // SLACK via curl
               withCredentials([string(credentialsId: 'SLACK_WEBHOOK', variable: 'SLACK_WEBHOOK_URL')]) {
                   sh """
                       curl -X POST -H 'Content-type: application/json' \\
                       --data '{
                           "text": "*Pipeline instable*\\n*Projet:* ${env.JOB_NAME}\\n*Build:* #${env.BUILD_NUMBER}\\n*URL:* ${env.BUILD_URL}",
                           "username": "Jenkins",
                           "icon_emoji": ":warning: "
                       }' "$SLACK_WEBHOOK_URL"
                   """
               }
           }
       }
   }



}
