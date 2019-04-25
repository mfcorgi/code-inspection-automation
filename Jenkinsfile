pipeline {
  agent any
  stages {
    stage('checkout') {
      parallel {
        stage('code inspection tools') {
          steps {
            dir(path: "${env.code_inspection_folder}") {
              git(url: "${env.code_inspection_url}", branch: "${env.code_inspection_branch}", credentialsId: "${env.code_inspection_credentials_id}", changelog: true)
              sh 'sudo rm -rf inspection'
            }

          }
        }
        stage('checkout source repository') {
          steps {
            dir(path: "${env.source_folder}") {
              git(url: "${params.repository}", branch: "${params.branch}", credentialsId: "${params.credentials_id}", changelog: true)
              sh 'sudo rm -rf inspection'
            }

          }
        }
      }
    }
    stage('install churn') {
      steps {
        dir(path: "${env.code_inspection_folder}") {
          sh '''#!/bin/bash -e

sudo cp sh/git-churn /usr/local/bin
sudo chmod u+x /usr/local/bin/git-churn
'''
        }

      }
    }
    stage('execute prepare.sh') {
      steps {
        sh '''#!/bin/bash -e

sudo chmod +x $code_inspection_folder/sh/jenkins/prepare.sh
sudo ./$code_inspection_folder/sh/jenkins/prepare.sh $code_inspection_folder $source_folder $code_climate_config'''
      }
    }
    stage('collect metrics') {
      parallel {
        stage('execute code climate') {
          steps {
            sh '''#!/bin/bash -e

sudo chmod +x $code_inspection_folder/sh/jenkins/execute_code_climate.sh
sudo $code_inspection_folder/sh/jenkins/execute_code_climate.sh "$PWD/$source_folder" '''
          }
        }
        stage('execute churn lifetime') {
          steps {
            sh '''#!/bin/bash -e

sudo chmod +x $code_inspection_folder/sh/jenkins/execute_churn_lifetime.sh
sudo $code_inspection_folder/sh/jenkins/execute_churn_lifetime.sh "$PWD/$code_inspection_folder" "$PWD/$source_folder"'''
          }
        }
        stage('execute churn recent') {
          steps {
            sh '''#!/bin/bash -e

sudo chmod +x $code_inspection_folder/sh/jenkins/execute_churn_recent.sh
sudo $code_inspection_folder/sh/jenkins/execute_churn_recent.sh "$PWD/$code_inspection_folder" "$PWD/$source_folder"'''
          }
        }
      }
    }
    stage('consolidate report') {
      steps {
        sh '''#!/bin/bash -e

workspace="$(realpath .)"
tools="$workspace/$code_inspection_folder" 
repo="$workspace/$source_folder"
outdir="$repo/inspection"

cd "$tools/script"

export PATH=$PATH:/home/ec2-user/.rbenv/plugins/ruby-build/bin:/home/ec2-user/.rbenv/shims:/home/ec2-user/.rbenv/bin:/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/home/ec2-user/.local/bin:/home/ec2-user/bin
ruby -v
bundle install
bundle exec exe/metrics-parser --dir=$outdir

echo "Wrote $outdir/data.csv"'''
      }
    }
    stage('save artifacts') {
      steps {
        dir(path: "${env.source_folder}") {
          archiveArtifacts 'inspection/*'
        }

      }
    }
    stage('send email') {
      steps {
        emailext(subject: 'Code Inspection Result', attachLog: true, to: 'maira@corgibytes.com', compressLog: true, from: 'jenkins@corgibytes.com', attachmentsPattern: '*', body: 'Code Inspection Result')
      }
    }
  }
  environment {
    source_folder = 'source-repository'
    code_inspection_folder = 'code-inspection-tools'
    code_inspection_url = 'https://github.com/corgibytes/code_inspection_tools.git'
    code_inspection_branch = 'master'
    code_inspection_credentials_id = 'github-credentials'
  }
  parameters {
    string(name: 'repository', defaultValue: 'https://github.com/corgibytes/ein-slackbot.git', description: 'Repository URL to inspect')
    string(name: 'branch', defaultValue: 'master', description: 'Branch that you want to inspect')
    string(name: 'credentials_id', defaultValue: 'github-credentials', description: 'Credentials ID configured in Jenkins that allows access to the repository')
    text(name: 'code_climate_config', defaultValue: '', description: 'Code Climate Configuration. If empty, will use the default code climate configuration from code inspection repository')
  }
}