pipeline {
  agent any
  stages {
    stage('checkout inspection tools') {
      parallel {
        stage('code inspection tools') {
          steps {
            git(branch: 'master', url: 'https://github.com/corgibytes/code_inspection_tools.git', credentialsId: 'github-credentials')
          }
        }
        stage('error') {
          steps {
            git(branch: 'master', credentialsId: 'github-credentials', url: 'https://github.com/corgibytes/ein-slackbot.git')
          }
        }
      }
    }
  }
}