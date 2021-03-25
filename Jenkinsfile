pipeline {
  agent any
  stages {
    stage('Show Variables') {
      steps {
        sh 'sh \'printenv\''
      }
    }

  }
  environment {
    REPO_NAME = 'Zygo.RuleEngine.UI'
  }
}