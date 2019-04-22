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
          sh '''sudo cp sh/git-churn /usr/local/bin
sudo chmod u+x /usr/local/bin/git-churn
'''
        }

      }
    }
    stage('run prepare.sh') {
      steps {
        sh '''#!/bin/bash

tools="$(realpath "$(dirname "$0")"/..)/code-inspection-tools"
repo="source-repository"
if [ -z "$repo" ]; then
    repo=$(realpath .)
fi
outdir="$(realpath "$repo")/inspection"


if [ -z "$repo" ]; then
    echo "Usage: $0 repo-to-analyze" 1>&2
    exit 1
fi

if [ ! -d "$repo/.git" ]; then
    echo "$repo does not appear to be a Git repository" 1>&2
    exit 1
fi

cd "$repo"

echo "Creating $outdir to store results."
mkdir -p "$outdir"

if [ ! -f "$outdir/cc-test-reporter" ]; then
    echo "Downloading CodeClimate tools."
    if [ "$(uname)" = Darwin ]; then
        url=https://codeclimate.com/downloads/test-reporter/test-reporter-latest-darwin-amd64
    elif [ "$(uname)" = Linux ]; then
        url=https://codeclimate.com/downloads/test-reporter/test-reporter-latest-linux-amd64
    else
        url=
    fi
    if [ "$url" = "" ]; then
        echo "Unable to automatically download CodeClimate tools.  Please see"
        echo "https://codeclimate.com/downloads/test-reporter/test-reporter-latest-darwin-amd64"
    else
        wget $url -O "$outdir/cc-test-reporter"
        chmod a+x "$outdir/cc-test-reporter"
    fi
fi

echo "Preparing CodeClimate configurations."
GLOBIGNORE=.
cp -v "$tools"/config/codeclimate/code_inspections/* .
shopt -u dotglob

if git ls-files --error-unmatch .eslintignore >& /dev/null ; then
    echo "* Using customer .eslintignore instead of our own."
    git checkout -- .eslintignore >& /dev/null
fi
if [ -f .eslintrc.js ] || [ -f .eslintrc.yaml ] || [ -f .eslintrc.json ] || [ -f .eslintrc ]; then
    echo "*Both a customer and a Corgibytes ESLint configuration are present.  See"
    echo "  https://eslint.org/docs/user-guide/configuring#configuration-file-formats"
fi

echo "Done. Please customize CodeClimate configurations as needed."'''
      }
    }
  }
  environment {
    source_repository = 'https://github.com/corgibytes/ein-slackbot.git'
    chunk_months_recent = '3'
  }
}