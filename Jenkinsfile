node {
    stage ("Checkout") {
        checkout scm
    }

    stage ("Integrate") {
        if (BRANCH_NAME == "master") {
            println "Already on master. No need to integrate."
        } else if (BRANCH_NAME.contains("hotfix/")) {
            println "Branch is a hotfix, not merging with master."
        } else {
            sh "git checkout master"
            sh "git merge origin/${BRANCH_NAME}"
        }
    }

    stage ("Build") {
        sh "./build.sh"
    }

    stage ("Publish") {
        if (BRANCH_NAME.contains("ready/")) {
            withCredentials([[
                        $class: 'UsernamePasswordMultiBinding',
                        credentialsId: "praqma-thi",
                        passwordVariable: 'PASSWORD',
                        usernameVariable: 'USERNAME']]) {
                sh "git push https://${USERNAME}:${PASSWORD}@github.com/praqma-thi/wf-test.git master"
                sh "git push https://${USERNAME}:${PASSWORD}@github.com/praqma-thi/wf-test.git :${BRANCH_NAME}"
            }
        } else {
            println "Not a 'ready' branch, not publishing results"
        }
    }

    stage ("Deploy to test") {
        sh "./deploy.sh test-server"
    }
}
