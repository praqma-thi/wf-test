boolean deploy = BRANCH_NAME.contains("hotfix/") || BRANCH_NAME == "master"
boolean integrate = !deploy
boolean publish = BRANCH_NAME.contains("ready/")

println ("----------------")
println "Branch: ${BRANCH_NAME}"
println "integrate: ${integrate}"
println "publish: ${publish}"
println "deploy: ${deploy}"
println ("----------------")

node ('master') {
    stage ("Checkout") {
        checkout scm
    }

    if (integrate) {
        stage ("Integrate") {
            sh "git checkout master"
            sh "git merge origin/${BRANCH_NAME}"
        }
    }

    stage ("Build") {
        sh "./build.sh"
    }

    if (publish) {
        stage ("Publish") {
            withCredentials([[
                $class: 'UsernamePasswordMultiBinding', 
                credentialsId: "praqma-thi", 
                passwordVariable: 'PASSWORD', 
                usernameVariable: 'USERNAME'
            ]]) {
                sh "git push https://${USERNAME}:${PASSWORD}@github.com/praqma-thi/wf-test.git master"
                sh "git push https://${USERNAME}:${PASSWORD}@github.com/praqma-thi/wf-test.git :${BRANCH_NAME}"
            }
        }
    }
}


if (deploy) {
    stage('Deploy?'){
        try {
            timeout(time: 1, unit: 'HOURS') {
                input 'Promote? (Remember to tag!)'
            }
        } catch (Throwable t) {
            currentBuild.result = 'SUCCESS'
            return
        }
    }

    node ('master') {
        stage ("Deploy to test") {
            sh "./deploy.sh test-server"
        }
    }
}
