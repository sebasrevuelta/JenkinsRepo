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
    sh '''
      docker pull semgrep/semgrep

      CONTAINER_NAME="semgrep-main-${BUILD_NUMBER}"

      echo "Starting Semgrep full scan on main branch..."
      echo "Container name: $CONTAINER_NAME"

      (
        while true; do
          docker stats --no-stream \
            --format "Semgrep memory: {{.MemUsage}} | CPU: {{.CPUPerc}}" \
            "$CONTAINER_NAME" 2>/dev/null || true
          sleep 5
        done
      ) &
      STATS_PID=$!

      docker run --name "$CONTAINER_NAME" --rm \
        -e SEMGREP_APP_TOKEN=$SEMGREP_APP_TOKEN \
        -e SEMGREP_REPO_NAME=$SEMGREP_REPO_NAME \
        -v "$(pwd):$(pwd)" --workdir $(pwd) \
        semgrep/semgrep semgrep ci

      SCAN_EXIT_CODE=$?

      kill $STATS_PID 2>/dev/null || true

      exit $SCAN_EXIT_CODE
    '''
  }
}
