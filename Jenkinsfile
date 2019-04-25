pipeline {
  agent any
  stages {
    stage('checkout') {
      parallel {
        stage('code inspection tools') {
          steps {
            dir(path: "${env.code_inspection_folder}") {
              git(url: "${env.code_inspection_url}", branch: "${env.code_inspection_branch}", credentialsId: "${env.code_inspection_credentials_id}", changelog: true)
              sh 'rm -rf inspection'
            }

          }
        }
        stage('checkout source repository') {
          steps {
            dir(path: "${env.source_folder}") {
              git(url: "${params.repository}", branch: "${params.branch}", credentialsId: "${params.credentials_id}", changelog: true)
              sh 'rm -rf inspection'
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
        dir(path: '"${env.code_inspection_folder}"') {
          sh '''#!/bin/bash -e

./sh/jenkins/prepare.sh $code_inspection_folder $source_folder'''
        }

      }
    }
    stage('collect metrics') {
      parallel {
        stage('execute code climate') {
          steps {
            dir(path: "${env.source_folder}") {
              sh '''#!/bin/bash -e

repo="$(realpath .)"
echo "Repository Path $repo"
outdir="inspection"

if [ ! -d ".git" ]; then
    echo "It does not appear to be a Git repository" 1>&2
    exit 1
fi

if [ ! -d "$outdir" ]; then
    echo "$outdir does not exist.  Please run prepare.sh" 1>&2
    exit 1
fi

if ! command -v codeclimate >& /dev/null; then
    echo "\'codeclimate\' is not runnable.  Please download and install the " 1>&2
    echo "CodeClimate CLI, including the wrapper package, as described here:" 1>&2
    echo https://github.com/codeclimate/codeclimate#code-climate-cli 1>&2
    exit 1
fi

cc="$outdir/ci_cc_results.json"
if [ ! -f "$cc" ]; then
    echo "Processing CodeClimate..."
    sudo docker pull codeclimate/codeclimate
    sudo docker run --env CODECLIMATE_DEBUG=1 --env CODECLIMATE_CODE="${repo}" --volume "${repo}":/code --volume /var/run/docker.sock:/var/run/docker.sock --volume /tmp/cc:/tmp/cc codeclimate/codeclimate analyze -f json > "$cc.new"
    mv "$cc.new" "$cc"    
fi
'''
            }

          }
        }
        stage('execute churn lifetime') {
          steps {
            dir(path: "${env.source_folder}") {
              sh '''#!/bin/bash -e

outdir="inspection"

export PATH="$tools/sh":$PATH

if [ ! -d ".git" ]; then
    echo "It does not appear to be a Git repository" 1>&2
    exit 1
fi

if [ ! -d "$outdir" ]; then
    echo "$outdir does not exist.  Please run prepare.sh" 1>&2
    exit 1
fi

if ! command -v codeclimate >& /dev/null; then
    echo "\'codeclimate\' is not runnable.  Please download and install the " 1>&2
    echo "CodeClimate CLI, including the wrapper package, as described here:" 1>&2
    echo https://github.com/codeclimate/codeclimate#code-climate-cli 1>&2
    exit 1
fi

lifetime="$outdir/churn_lifetime.txt"

if [ ! -f "$lifetime" ]; then
    echo "Processing lifetime churn..."
    git churn > "$lifetime.new"
    mv "$lifetime.new" "$lifetime"
fi
'''
            }

          }
        }
        stage('execute churn recent') {
          steps {
            dir(path: "${env.source_folder}") {
              sh '''#!/bin/bash -e

outdir="inspection"

export PATH="$tools/sh":$PATH

if [ ! -d ".git" ]; then
    echo "It does not appear to be a Git repository" 1>&2
    exit 1
fi

if [ ! -d "$outdir" ]; then
    echo "$outdir does not exist.  Please run prepare.sh" 1>&2
    exit 1
fi

if ! command -v codeclimate >& /dev/null; then
    echo "\'codeclimate\' is not runnable.  Please download and install the " 1>&2
    echo "CodeClimate CLI, including the wrapper package, as described here:" 1>&2
    echo https://github.com/codeclimate/codeclimate#code-climate-cli 1>&2
    exit 1
fi


recent="$outdir/churn_recent.txt"
if [ ! -f "$recent" ]; then
    echo "Processing recent churn..."
     git churn --since=\'3 months ago\' > "$recent.new"
     mv "$recent.new" "$recent"
fi
'''
            }

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