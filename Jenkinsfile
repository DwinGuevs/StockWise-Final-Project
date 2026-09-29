pipeline {
  agent any

  options {
    timestamps()
    disableConcurrentBuilds()
  }

  environment {
    TAG = "${BUILD_NUMBER}"
  }

  triggers {
    pollSCM('H/2 * * * *')
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Test') {
      steps {
        sh '''
          set -e

          echo "===== Testing Product API ====="
          docker build --target test -t stockwise/product-api:test ./product-api

          echo "===== Testing Inventory API ====="
          docker build --target test -t stockwise/inventory-api:test ./inventory-api

          echo "===== Testing Sales API ====="
          docker build --target test -t stockwise/sales-api:test ./sales-api

          echo "===== ALL TESTS PASSED ====="
        '''
      }
    }

    stage('Build') {
      steps {
        sh 'docker compose build'
      }
    }

    stage('Deploy') {
      steps {
        sh 'docker compose up -d --no-build --remove-orphans'
      }
    }

    stage('Smoke Test') {
      steps {
        sh '''
          echo "===== Starting Smoke Test ====="

          for i in $(seq 1 30); do
            if curl -fsS http://localhost:8080/api/products >/dev/null && \
               curl -fsS http://localhost:8080/api/inventory >/dev/null && \
               curl -fsS http://localhost:8080/api/sales >/dev/null; then

              echo "===== SMOKE TEST PASSED ====="
              exit 0
            fi

            echo "Waiting for StockWise services..."
            sleep 3
          done

          echo "===== SMOKE TEST FAILED ====="
          exit 1
        '''
      }
    }
  }

  post {
    success {
      echo 'StockWise pipeline succeeded.'
    }

    failure {
      echo 'Pipeline failed. Check the failed stage and service logs.'
    }

    always {
      sh 'docker image prune -f || true'
      echo 'Pipeline finished.'
    }
  }
}