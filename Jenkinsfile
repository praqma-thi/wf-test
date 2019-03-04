pipeline {
  options { buildDiscarder(logRotator(numToKeepStr: "5")) }

  triggers { pollSCM("H/2 * * * *") }

  agent {
    node {
      label "master"
    }
  }

  stages {
    stage ("Checkout") {
      steps {
        checkout scm
      }
    }

    stage ("Integrate") {
      when {
        expression {
          return env.BRANCH_NAME != "master" && !env.BRANCH_NAME =~ /hotfix\/.*/
        }
      }

      steps {
        withCredentials([sshUserPrivateKey(credentialsId: "bitbucket-ssh", keyFileVariable: "BB_KEY")]) {
          sh "echo ssh -i ${BB_KEY} -l git \\"\\$@\\" > with_ssh.sh"
          sh "chmod +x with_ssh.sh"
          withEnv(["GIT_SSH=with_ssh.sh"]) {
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
      }
    }

    stage ("Build") {
      steps {
          sh "sh build.sh"
      }
    }

    stage ("Publish") {
      when {
        expression {
          return env.BRANCH_NAME =~ /ready\/.*/
        }
      }

      steps {
        withCredentials([sshUserPrivateKey(credentialsId: "bitbucket-ssh", keyFileVariable: "BB_KEY")]) {
          sh "echo ssh -i ${BB_KEY} -l git \\"\\$@\\" > with_ssh.sh"
          sh "chmod +x with_ssh.sh"
          withEnv(["GIT_SSH=with_ssh.sh"]) {
            sh "git push origin master"
            sh "git push origin :${env.BRANCH_NAME}"
          }
        }
      }
    }


    stage ("Deploy to test") {
      steps {
        withCredentials([sshUserPrivateKey(credentialsId: "deployment-ssh", keyFileVariable: "DEP_KEY")]) {
          sh "sh deploy.sh ${DEP_KEY}"
        }

        withCredentials([sshUserPrivateKey(credentialsId: "bitbucket-ssh", keyFileVariable: "BB_KEY")]) {
          sh "echo ssh -i ${BB_KEY} -l git \\"\\$@\\" > with_ssh.sh"
          sh "chmod +x with_ssh.sh"
          withEnv(["GIT_SSH=with_ssh.sh"]) {
            sh "git tag -f current-dev"
            sh "git push -f origin current-dev"
          }
        }
      }
    }
  }
}
