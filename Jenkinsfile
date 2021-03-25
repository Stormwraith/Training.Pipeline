pipeline {

  environment {
    REPO_NAME = 'Training.Pipeline'
    BUILD_MAJOR = sh(returnStdout: true, script: "echo \$(date +%y)").trim();
    BUILD_MINOR = sh(returnStdout: true, script: "echo \$(((\$(date +%-m)-1)/3+1))").trim();
    BUILD_PATCH = sh(returnStdout: true, script: "echo \$(date +%m%d)").trim();
    BUILD_REV = 0;
  }

  agent { label 'linux' }

  stages {
    stage('Show Variables') {
      steps {
        sh 'printenv'
      }
    }

    stage('Build') {
    
      agent { docker { image 'node:latest' } }

      stages {
        
        stage('NPM Install') {
          steps { sh 'npm install' }
        }

        stage('Run Tests') {
          parallel {
            stage('Static code analysis') {
                steps { sh 'npm run-script lint' }
            }
            stage('Unit tests') {
                steps { sh 'npm run-script test' }
            }
          }
        }

        stage('Build') {
          steps { sh 'npm run-script build' }
        }
      }
    }
  }
}