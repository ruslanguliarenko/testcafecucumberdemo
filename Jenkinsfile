#!/usr/bin/env groovy
pipeline {
  agent {
    docker {
      image 'browsers'
    }
  }

  environment {
    GIT_SSL_NO_VERIFY = true
  }

  stages {
    stage('Build and run e2e tests ') {
      steps {

        echo "Building Job with ID ${env.BUILD_ID}"

        sh 'mkdir -p reports'
       sh 'npm install'
       sh 'npm run e2edocker'
      }
    }

    stage('Publish reports') {
      steps {
        echo "Publishing reports: "
        script{
          Date date = new Date()
          String datePart = date.format("dd/MM/yyyy")
          String timePart = date.format("HH:mm:ss")
          def DATETIME = '_' + datePart + '_' + timePart
          publishHTML target: [allowMissing: false, alwaysLinkToLastBuild: false, keepAll: true, reportDir: 'reports/combined', reportFiles: 'index.html', reportName: 'HTML Report'+ DATETIME, reportTitles: '']
        }
      }
    }
    stage('Cleaning up') {
      steps{
        script {
          def result = sh script: 'node src/checkTests.js', returnStatus: true
          if (result != 0) {

            echo 'Check log for failed e2e tests!'
            currentBuild.result = 'FAILURE'
            return
          }
        }
      }
    }
  }
  post {
    failure {
      script{
        def JOBURL = env.JOB_URL
        if(JOBURL == null){
          JOBURL = "WARNING: Jenkins URL not set! Set Jenkins URL at http://localhost:8080/configure"
          echo "$JOBURL \n\nERROR: Build failed! Check CucumberJS reports for more details\n"
        }
        else{
          echo "ERROR: Build failed! Check the CucumberJS reports at $JOBURL"
        }
      }
    }
  }
}