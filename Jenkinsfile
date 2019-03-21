pipeline {
  options { buildDiscarder(logRotator(numToKeepStr: "10")) }
  triggers { pollSCM("H/1 * * * *") }
  agent none

  stages {
    stage ("Integrate") {
      agent { label "master" }
      when {
        expression {
          // Don't merge master or hotfix branches
          return !(env.BRANCH_NAME == "master" || env.BRANCH_NAME =~ /hotfix\/.*/)
        }
      }

      steps {
        // Fetch latest master
        sh "git remote set-branches --add origin master"
        sh "git fetch"

        // Rebase current branch on top of master
        sh "git checkout -b ${env.BRANCH_NAME}"
        sh "git rebase origin/master"

        // Merge changes into master
        sh "git checkout master"
        sh "git merge --ff-only ${env.BRANCH_NAME}"
      }
    }

    stage ("Build") {
      agent { label "master" }
      steps {
        sh "echo 'Add any build and/or verification steps here'"
      }
    }

    stage ("Publish") {
      agent { label "master" }
      when {
        expression {
          // Only publish merges made with ready branches
          return env.BRANCH_NAME =~ /ready\/.*/
        }
      }

      steps {
        sh "git push origin master"
        sh "git push origin :${env.BRANCH_NAME}"
      }
    }

    // Ask if we want to deploy to the test environment
    stage("Deploy to test?") {
      agent none
      steps {
        script {
          try {
            timeout(time: 60, unit: 'MINUTES') {
              env.DEPLOY = input message: 'Deploy to test?',
              parameters: [choice(name: 'Deploy to test?', choices: 'yes\nno', description: 'Choose "yes" if you want to deploy this build')]
            }
          } catch (err) {
            println "Not deploying."
          }
        }
      }
    }

    stage ("Deploy to test") {
      agent { label "master" }
      steps {
        withCredentials([sshUserPrivateKey(credentialsId: "deploy", keyFileVariable: "DEP_KEY")]) {
          sh "sh deploy.sh test ${DEP_KEY}"
        }

        sh "git tag -f current-dev"
        sh "git push -f origin current-dev"
      }
    }

    stage("Release?") {
      agent none
      steps {
        script {
          try {
            timeout(time: 60, unit: 'MINUTES') {
              env.RELEASE_TAG = input message: 'Specify the release tag',
                parameters: [
                  string(defaultValue: '', description: 'The release tag (e.g.: 1.0.0)', name: 'Release tag')
                ]
            }
          } catch (err) {
            println "Not releasing."
          }
        }
      }
    }

    stage("Release") {
      agent { label "master" }
      when {
        expression {
          return env.RELEASE_TAG && env.RELEASE_TAG.trim() != ""
        }
      }

      steps {
        sh "git tag ${env.RELEASE_TAG}"

        withCredentials([sshUserPrivateKey(credentialsId: "deploy", keyFileVariable: "DEP_KEY")]) {
          sh "sh deploy.sh release ${DEP_KEY}"
        }

        sh "git push origin ${env.RELEASE_TAG}"

        sh "git tag -f current-rel"
        sh "git push -f origin current-rel"
      }
    }
  }
}
