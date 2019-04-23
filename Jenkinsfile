pipeline {
  agent any
  stages {
    stage('checkout') {
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
    stage('install tools') {
      parallel {
        stage('install churn') {
          steps {
            dir(path: 'code-inspection-tools') {
              sh '''sudo cp sh/git-churn /usr/local/bin
sudo chmod u+x /usr/local/bin/git-churn
'''
            }

          }
        }
        stage('install code climate') {
          steps {
            sh '''curl -L https://github.com/codeclimate/codeclimate/archive/master.tar.gz | tar xvz
cd codeclimate-* && sudo make install'''
          }
        }
      }
    }
    stage('execute prepare.sh') {
      steps {
        sh '''#!/bin/bash

tools="$(realpath code-inspection-tools)"
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
    stage('execute run.sh') {
      steps {
        sh '''#!/bin/bash

tools="$(realpath code-inspection-tools)"
repo="source-repository"
if [ -z "$repo" ]; then
    repo=$(realpath .)
fi
outdir="$(realpath "$repo")/inspection"

export PATH="$tools/sh":$PATH

if [ -z "$repo" ]; then
    echo "Usage: $0 repo-to-analyze" 1>&2
    exit 1
fi

if [ ! -d "$repo/.git" ]; then
    echo "$repo does not appear to be a Git repository" 1>&2
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

cd "$repo"

lifetime="$outdir/churn_lifetime.txt"
if [ ! -f "$lifetime" ]; then
    echo "Processing lifetime churn..."
    git churn > "$lifetime.new"
    mv "$lifetime.new" "$lifetime"
fi

recent="$outdir/churn_recent.txt"
if [ ! -f "$recent" ]; then
    echo "Processing recent churn..."
     git churn --since=\'3 months ago\' > "$recent.new"
     mv "$recent.new" "$recent"
fi

cc="$outdir/ci_cc_results.json"
if [ ! -f "$cc" ]; then
    echo "Processing CodeClimate..."
    codeclimate analyze -f json > "$cc.new"

    # If Python is available, then use it to pretty-print the JSON,
    # to aid in manual inspection.
    if command -v python >& /dev/null ; then
        python -m json.tool "$cc.new" > "$cc"
        rm "$cc.new"
    else
        mv "$cc.new" "$cc"
    fi
fi


(cd "$tools/script" && bundle exec metrics-parser --dir="$outdir")
echo "Wrote $outdir/data.csv"'''
      }
    }
  }
  environment {
    source_repository = 'https://github.com/corgibytes/ein-slackbot.git'
  }
}