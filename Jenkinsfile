pipeline {
    agent any

    stages {
        stage('Preparation') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: 'main']],
                    doGenerateSubmoduleConfigurations: false,
                    extensions: [[$class: 'CloneOption', depth: 0, noTags: false, reference: '', shallow: true]],
                    submoduleCfg: [],
                    userRemoteConfigs: [[url: 'git@github.com:mkhaufillah/flow.git', credentialsId: 'github-mkhaufillah']]
                ])
                sh 'ls --all'
        //         sh 'if [[ "$(uname)" != *"Linux"* ]]; then exit 1; fi'
        //         sh 'if ! which unzip > /dev/null; then sudo apt update && sudo apt install -y unzip; fi'
        //         sh 'if ! which bun > /dev/null; then sudo curl -fsSL https://bun.sh/install | bash; fi'
        //         sh 'sudo bun install'
            }
        }

        stage('Deploy Flow Bun') {
            steps {
                withCredentials([string(credentialsId: 'deployment-key', variable: 'DEPLOYMENT_KEY')]) {
                    sh 'sudo echo $DEPLOYMENT_KEY'
        //             sh 'sudo cp -f systemd/flow-bun.service /lib/systemd/system/flow-bun.service'
                }
            }
        }
    }
}