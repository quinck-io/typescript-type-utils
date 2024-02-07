import hudson.tasks.junit.TestResultSummary

String lastRunningStage; 

pipeline {
  agent { label 'master' }

  tools {
    nodejs "node20"
  }

  options {
    disableConcurrentBuilds()
  }
  
  // environment {
  //   // DISCORD_WEBHOOK_URL = ''
  // }

  stages {

    stage('Project Init') {
      steps {
        script {
          lastRunningStage="Project Init"
        }
        sh 'npm ci'
      }
    }
    
    stage("Lint") {
      steps {
          script {
            lastRunningStage="Lint"
          }
          sh 'npm run lint'
      }
    }

    stage("Build") {
      when {
        anyOf {
          branch 'dev';
          branch 'master'
        }
      }
      steps {
        script {
          lastRunningStage = 'Build'
        }
        sh 'npm run build'
      }
    }

    stage("Publish") {
      when {
        anyOf {
          branch 'master'
        }
      }
      steps {
        script {
          lastRunningStage = 'Publish'
          withCredentials([string(credentialsId: 'quinck-npm-token', variable: 'NPM_TOKEN')]) {
            sh '''
              set +x
              echo "//registry.npmjs.org/:_authToken=${NPM_TOKEN}" > .npmrc
              npm whoami
              
              PUBLISHED_VERSION=$(npm show @quinck/type-utils version || echo "NONE")
              PACKAGE_VERSION=$(cat package.json | grep version | head -1 | awk -F: '{ print $2 }' | sed 's/[\",]//g' | tr -d '[[:space:]]')
              if [ "${PUBLISHED_VERSION}" = "${PACKAGE_VERSION}" ]; then
                echo "The current package version has already been published"
              else
                echo "Do pubblication"
                npm publish --access public
              fi
              
              rm .npmrc
            '''
          }
        }
      }
    }
  }

  post {
    always {
      script {
        // notifyDiscord(currentBuild.result, lastRunningStage, testResult)
        deleteDir() /* clean up our workspace */
        }
    }
    failure {
      script {
        deleteDir() /* clean up our workspace */
        sh 'exit 1'
      }
    }
  }
}