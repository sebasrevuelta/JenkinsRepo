pipeline {
  agent any
  environment {
    // Required for a Semgrep AppSec Platform-connected scan:
    SEMGREP_APP_TOKEN = credentials('SEMGREP_APP_TOKEN')
    // Set repo name to expected format
    SEMGREP_REPO_NAME = env.GIT_URL.replaceFirst(/^https:\/\/github.com\/(.*)$/, '$1')
  }
  stages {
    stage('print-branch') {
      steps {
        echo "BRANCH_NAME: ${env.BRANCH_NAME}"
        echo "GIT_BRANCH: ${env.GIT_BRANCH}"
        echo "CHANGE_BRANCH: ${env.CHANGE_BRANCH}"
        echo "CHANGE_TARGET: ${env.CHANGE_TARGET}"
      }
    }

    stage('semgrep-diff-scan') {
      when {
        branch "PR-*"
      }
      steps {
        sh '''git fetch --no-tags --force --progress -- $GIT_URL +refs/heads/$CHANGE_TARGET:refs/remotes/origin/$CHANGE_TARGET
              git checkout -b $CHANGE_TARGET origin/$CHANGE_TARGET
              git checkout $GIT_BRANCH
           '''
        sh '''docker pull semgrep/semgrep && \
            docker run \
            -e SEMGREP_APP_TOKEN=$SEMGREP_APP_TOKEN \
            -e SEMGREP_REPO_NAME=$SEMGREP_REPO_NAME \
            -e SEMGREP_BASELINE_REF=$(git merge-base $GIT_BRANCH $CHANGE_TARGET) \
            -e SEMGREP_PR_ID="${env.CHANGE_ID}"
            -v "$(pwd):$(pwd)" --workdir $(pwd) \
            semgrep/semgrep semgrep ci '''
      }
    }

    stage('semgrep-scan') {
      when {
        anyOf {
          branch "main"
          expression { env.BRANCH_NAME == 'main' }
          expression { env.GIT_BRANCH == 'origin/main' }
          expression { env.GIT_BRANCH == 'main' }
        }
      }
      steps {
        //sh '''docker pull semgrep/semgrep && \
        //    docker run \
        //    -e SEMGREP_APP_TOKEN=$SEMGREP_APP_TOKEN \
        //    -e SEMGREP_REPO_NAME=$SEMGREP_REPO_NAME \
        //    -v "$(pwd):$(pwd)" --workdir $(pwd) \
        //    semgrep/semgrep semgrep ci '''
      }
    }
  }

  stage('semgrep-docker-memory') {
    steps {
      sh '''
        set -e
  
        CONTAINER_NAME="semgrep-memory-test-$BUILD_NUMBER"
  
        docker pull semgrep/semgrep
  
        docker run --name "$CONTAINER_NAME" \
          -e SEMGREP_APP_TOKEN="$SEMGREP_APP_TOKEN" \
          -e SEMGREP_REPO_NAME="$SEMGREP_REPO_NAME" \
          -v "$PWD:/src" \
          -w /src \
          semgrep/semgrep \
          semgrep ci &
  
        SEMGREP_PID=$!
  
        echo "===== DOCKER MEMORY WHILE SEMGREP RUNS ====="
        while docker ps --format '{{.Names}}' | grep -q "$CONTAINER_NAME"; do
          docker stats --no-stream "$CONTAINER_NAME" || true
          sleep 5
        done
  
        wait $SEMGREP_PID
      '''
    }
  }
  
  post {
    always {
      cleanWs()
    }
  }
}
