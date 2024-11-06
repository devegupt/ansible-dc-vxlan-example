pipeline {
    agent any
    // agent {
    //     docker {
    //         image 'danischm/nac:0.1.4'
    //         label 'digidev'
    //         args '-u root'
    //     }
    // }

    environment {
        ND_HOST = credentials('ND_HOST')
        ND_DOMAIN = credentials('ND_DOMAIN')
        ND_USERNAME = credentials('ND_USERNAME')
        ND_PASSWORD = credentials('ND_PASSWORD')
        NDFC_SW_USERNAME = credentials('NDFC_SW_USERNAME')
        NDFC_SW_PASSWORD = credentials('NDFC_SW_PASSWORD')
        // WEBEX_TOKEN = credentials('WEBEX_TOKEN')
        // WEBEX_ROOM_ID = ''
        // ND_HOST = 10.195.225.172
        // ND_DOMAIN = test-domain
        // ND_USERNAME = developer
        // ND_PASSWORD = C1sc0@123
        // NDFC_SW_USERNAME = admin
        // NDFC_SW_PASSWORD = C1sc0@123
    }

    options {
        disableConcurrentBuilds()
    }

    stages {
        stage('Setup') {
            steps {
                // sh 'pip install --upgrade pip'
                // sh 'pip install -r requirements.txt'
                sh 'git clone --depth 1 --branch master https://wwwin-github.cisco.com/devegupt/nac-vxlan.git'
// + git clone --depth 1 --branch master git@wwwin-github.cisco.com:netascode/nac-vxlan.git
// Cloning into 'nac-vxlan'...
// Host key verification failed.
// fatal: Could not read from remote repository
                sh 'mkdir -p collections/ansible_collections/cisco'
                sh 'ansible-galaxy collection install -p collections/ansible_collections/ -r requirements.yaml'
                sh 'git clone --depth 1 --branch develop https://github.com/devegupt/ansible-dc-vxlan.git collections/ansible_collections/cisco/nac_dc_vxlan'
                }
            }
        stage('Validate') {
            steps {
                // sh 'pwd |& tee pwd_output.txt'
                sh 'set -o pipefail && iac-validate host_vars/nac-ndfc1 -s nac-vxlan/schemas/schema.yaml -r collections/ansible_collections/cisco/nac_dc_vxlan/roles/validate/files/rules/ |& tee validate_output.txt'
            }
        }
        stage('Deploy') {
            steps {
                sh 'set -o pipefail && ansible-playbook -i inventory.yaml vxlan.yaml |& tee deploy_output.txt'
            }
        }
        stage('Test') {
            steps {
                sh 'set -o pipefail && iac-test -d host_vars/nac-ndfc1 -d nac-vxlan/defaults/defaults.yaml -f nac-vxlan/jinja_filters -t nac-vxlan/templates -o ./test_results |& tee test_output.txt'
            }
            post {
                always {
                    archiveArtifacts 'test_results/log.html, test_results/output.xml, test_results/report.html, test_results/xunit.xml'
                    junit 'test_results/xunit.xml'
                }
            }
        }
    }
    // post {
    //     always {
    //         sh "BUILD_STATUS=${currentBuild.currentResult} python3 .ci/webex-notification-jenkins.py"
    //         sh 'rm -rf *.txt *.yaml previous ${CONFIG_REPO_NAME} test_results rendered'
    //         cleanWs()
    //     }
    // }
}