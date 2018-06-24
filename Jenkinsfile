pipeline {
    agent {
      label "jenkins-maven"
    }
    environment {
      ORG               = 'affix'
      APP_NAME          = 'nextcloud'
      CHARTMUSEUM_CREDS = credentials('jenkins-x-chartmuseum')
    }
    stages {
      stage('CI Build and push snapshot') {
        when {
          branch 'PR-*'
        }
        environment {
          PREVIEW_VERSION = "0.0.0-SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER"
          PREVIEW_NAMESPACE = "$APP_NAME-$BRANCH_NAME".toLowerCase()
          HELM_RELEASE = "$PREVIEW_NAMESPACE".toLowerCase()
        }
        steps {
         container('maven') {
           sh 'cd nextcloud'
           sh "make preview"
           sh "jx preview --app $APP_NAME --dir ../.."
         }
        }
      }
      stage('Publish Chart') {
        when {
          branch 'master'
        }
        steps {
          dir('nextcloud') {
            container('maven') {
              sh "echo \$(jx-release-version) > VERSION"
              // release the helm chart
              sh 'make release'

              // promote through all 'Auto' promotion Environments
              sh 'jx promote -b --all-auto --timeout 1h --version \$(jx-release-version)'
            }
          }
        }
      }
    }
    post {
        always {
            cleanWs()
        }
        failure {
            input """Pipeline failed.
We will keep the build pod around to help you diagnose any failures.
Select Proceed or Abort to terminate the build pod"""
        }
    }
  }
