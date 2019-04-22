pipeline {
  agent any
  stages {
    stage('checkout inspection tools') {
      parallel {
        stage('code inspection tools') {
          steps {
            dir(path: 'code-inspection-tools') {
              git(url: 'https://github.com/corgibytes/code_inspection_tools.git', branch: 'master', credentialsId: 'github-credentials', changelog: true)
            }

          }
        }
        stage('checkout source repository') {
          steps {
            dir(path: 'source-repository') {
              git(url: "${env.source_repository}", branch: 'master', credentialsId: 'github-credentials', changelog: true)
            }

          }
        }
      }
    }
    stage('install churn') {
      steps {
        dir(path: 'code-inspection-tools') {
          sh '''echo $PATH
cp sh/git-churn /usr/local/bin
chmod u+x /usr/local/bin/git-churn'''
        }

      }
    }
  }
  environment {
    source_repository = 'https://github.com/corgibytes/ein-slackbot.git'
    chunk_months_recent = '3'
  }
}