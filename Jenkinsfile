pipeline {
  agent any
  stages {
    stage('code inspection tools') {
      steps {
        git(branch: 'master', url: 'https://github.com/corgibytes/code_inspection_tools.git', credentialsId: 'github-credentials')
      }
    }
  }
}