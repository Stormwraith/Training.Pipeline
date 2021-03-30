pipeline {

  environment {
    REPO_NAME = 'Training.Pipeline'
    BUILD_MAJOR = sh(returnStdout: true, script: "echo \$(date +%y)").trim();
    BUILD_MINOR = sh(returnStdout: true, script: "echo \$(((\$(date +%-m)-1)/3+1))").trim();
    BUILD_PATCH = sh(returnStdout: true, script: "echo \$(date +%m%d)").trim();
    BUILD_REV = 0;
    S3_BUCKET_NAME_PREFIX = 'storm-cicd-deploy-';
    AWS_REGION = 'eu-west-2';
  }

  agent { label 'linux' }

  stages {
    stage('Show Variables') {
      steps {
        sh 'printenv'
      }
    }

    stage('Build Start') {
    
      agent { docker { image 'node:latest' } }

      stages {
        
        stage('NPM Install') {
          steps { sh 'npm install --prefix ./Training-Pipeline-UI' }
        }

        stage('Run Tests') {
          // parallel {
            // stage('Static code analysis') {
                steps { sh 'npm run-script lint --prefix ./Training-Pipeline-UI' }
            // }
            // stage('Unit tests') {
            //     steps { sh 'npm run-script automated_test --prefix ./Training-Pipeline-UI' }                       
            // }
          // }
        }
        
        stage('Calculate Build No.'){
          stages{
          

            stage ('Determine Target Env.'){

              steps {
                script {

                  if(BRANCH_NAME.toLowerCase().startsWith('master')) {
                    env.DEPLOYMENT_ENVIRONMENT = 'stable';
                  } else if(BRANCH_NAME.toLowerCase().startsWith('qa')) {
                    env.DEPLOYMENT_ENVIRONMENT = 'qa';
                  }
                  else {
                    env.DEPLOYMENT_ENVIRONMENT = 'dev';
                  }

                  echo "Deploying to the ${DEPLOYMENT_ENVIRONMENT} environment."

                }
              }
            }

            stage('Set Revision from based on Git Tag') {
              steps {
                script {

                  def tagMarker = '';

                  if(BRANCH_NAME == 'master') {
                    tagMarker = '.';

                  } else {
                    tagMarker = '-CI';
                  }

                  def grepRegEx = "${BUILD_MAJOR}.${BUILD_MINOR}.${BUILD_PATCH}${tagMarker}\\K(\\d+\$)";

                  echo 'grepRegEx: ' +  grepRegEx;

                  def lastRevision = sh(returnStdout: true, script: "git tag -l --sort=v:refname | grep -oP '${grepRegEx}' | tail -1").trim();

                  echo 'lastRevision: ' +  lastRevision;

                  if (lastRevision) {
                    echo 'we have a value: ' + lastRevision;
                    BUILD_REV = lastRevision.toInteger() + 1;
                    echo 'BUILD_REV: ' + BUILD_REV;

                  } else {
                    echo 'we have no value';
                    BUILD_REV = 0;
                    echo 'BUILD_REV: ' + BUILD_REV;
                  }
                }
              }
            }

            stage('Set Version - Master Branch') {
              when {
                branch 'master'
              }
              steps {
                script {
                    env.BUILD_VERSION = sh(returnStdout: true, script: "echo ${BUILD_MAJOR}.${BUILD_MINOR}.${BUILD_PATCH}.${BUILD_REV}").trim()
                    echo 'BUILD_VERSION: ' + BUILD_VERSION
                    env.PACKAGE_VERSION = sh(returnStdout: true, script: "echo ${BUILD_MAJOR}.${BUILD_MINOR}.${BUILD_PATCH}.${BUILD_REV}").trim()
                    echo 'PACKAGE_VERSION: ' + PACKAGE_VERSION
                    //NPM pacakges for master are e.g. 21.1.0, 21.1.1, 21.1.2....21.1.999
                    //Packages for other branches are e.g. 21.1.0-rc.1...21.1.0-rc.99
                    env.NPM_BUILD_VERSION = sh(returnStdout: true, script: "echo ${BUILD_MAJOR}.${BUILD_MINOR}.${BUILD_ID}").trim()
                    echo 'NPM_BUILD_VERSION: ' + NPM_BUILD_VERSION

                }
              }
            }

            stage('Set Version - Other Branch') {
              when {
                  not {
                      branch 'master'
                  }
              }
              steps {
                script {
                    env.BUILD_VERSION = sh(returnStdout: true, script: "echo ${BUILD_MAJOR}.${BUILD_MINOR}.${BUILD_PATCH}.${BUILD_REV}").trim()
                    echo 'BUILD_VERSION: ' + BUILD_VERSION
                    env.PACKAGE_VERSION = sh(returnStdout: true, script: "echo ${BUILD_MAJOR}.${BUILD_MINOR}.${BUILD_PATCH}-CI${BUILD_REV}").trim()
                    echo 'PACKAGE_VERSION: ' + PACKAGE_VERSION
                    //NPM pacakges for master are e.g. 21.1.0, 21.1.1, 21.1.2....21.1.999
                    //Packages for other branches are e.g. 21.1.0-rc.1...21.1.0-rc.99
                    env.NPM_BUILD_VERSION = sh(returnStdout: true, script: "echo ${BUILD_MAJOR}.${BUILD_MINOR}.0-CI.${BUILD_ID}").trim()
                    echo 'NPM_BUILD_VERSION: ' + NPM_BUILD_VERSION
                }
              }
            }

            stage('Update Git Tag') {
              when{not{branch "PR-*"}}
              steps {
                echo 'PACKAGE_VERSION: ' + PACKAGE_VERSION
                //script {
                //    sh '''
                //      git tag v${PACKAGE_VERSION}
                //      git push --tags
                //    '''
                //}
              }
            }

          }
        }

        stage('Build') {
          steps { 
            sh 'npm run-script production_build --prefix ./Training-Pipeline-UI' 
          }
        }

        stage('Zip') {
          steps { 
            zip zipFile: "${REPO_NAME}.${PACKAGE_VERSION}.zip", archive: true, dir: './Training-Pipeline-UI/dist'
          }
        }

        
        stage('Upload to S3'){
          steps{
            withAWS(credentials:"aws_deploy", region:"${AWS_REGION}") {
              sh '''
                aws s3 cp ".//Training-Pipeline-UI//dist//Training-Pipeline-UI" "s3://${S3_BUCKET_NAME_PREFIX}${DEPLOYMENT_ENVIRONMENT}/${REPO_NAME}/${PACKAGE_VERSION}/" --recursive
              '''
            }
          }
        }

      }
    }
  }
}