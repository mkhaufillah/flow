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
                sh 'if [[ "$(uname)" != *"Linux"* ]]; then exit 1; fi'
                sh 'if ! which unzip > /dev/null; then sudo apt update && sudo apt install -y unzip; fi'
                sh 'if ! test -f /root/.bun/bin/bun; then sudo su && curl -fsSL https://bun.sh/install | bash; fi'
                sh 'sudo /root/.bun/bin/bun install'
            }
        }

        stage('Deploy Flow Bun') {
            steps {
                withCredentials([string(credentialsId: 'deployment-key', variable: 'DEPLOYMENT_KEY')]) {
                    sh 'sudo echo $DEPLOYMENT_KEY'
                    sh 'sudo cp -f systemd/flow-bun.service /lib/systemd/system/flow-bun.service'
                    sh 'systemctl daemon-reload'
                    sh 'systemctl restart flow-bun.service'
                    sh 'sudo cp -f nginx/flow.filla.id /etc/nginx/sites-available/flow.filla.id'
                    sh 'sudo nginx -t'
                    sh 'sudo systemctl restart nginx'
                }
            }
        }
    }
}