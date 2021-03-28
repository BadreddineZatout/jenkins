pipeline {
  agent any
  stages {
    stage('build') {
      post {
        failure {
          script {
            mail="Build failed"
          }

        }

        success {
          script {
            mail="Build succeeded"
          }

        }

      }
      steps {
        bat 'gradle build'
        bat 'gradle javadoc'
        archiveArtifacts 'build/libs/*.jar'
        junit(testResults: 'build/test-results/test/*.xml', allowEmptyResults: true)
      }
    }

    stage('Mail Notification') {
      steps {
        mail(subject: 'Jenkins notification', body: mail, cc: 'hb_zatout@esi.dz')
      }
    }

    stage('Code Analysis') {
      parallel {
        stage('Code Analysis') {
          steps {
           withSonarQubeEnv('My SonarQube Server') {
                bat 'gradle sonarqube'
           }
          }
        }
        stage("Quality Gate") {
            steps {
              timeout(time: 1, unit: 'HOURS') {
                waitForQualityGate abortPipeline: true
              }
            }
          }
        stage('Test Reporting') {
          steps {
            cucumber 'reports/*json'
          }
        }

      }
    }

    stage('Deployment') {
      steps {
        bat 'gradle publish'
      }
    }

    stage('Slack Notification') {
      steps {
        slackSend(token: 'T01LX7C6ZBR/B01SJQJ7LF7/76DVeKq505GAktJwLyewt5u6', baseUrl: 'https://hooks.slack.com/services/', channel: '#project', message: 'Project is built newly and deployed')
      }
    }

  }
  environment {
    mail = ''
  }
}