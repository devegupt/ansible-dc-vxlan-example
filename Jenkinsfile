pipeline {
    agent {
        docker {
            image 'danischm/nac:0.1.4'
            label 'digidev'
            args '-u root'
        }
    }

    environment {
        ND_HOST = credentials('ND_HOST')
        ND_DOMAIN = credentials('ND_DOMAIN')
        ND_USERNAME = credentials('ND_USERNAME')
        ND_PASSWORD = credentials('ND_PASSWORD')
        NDFC_SW_USERNAME = credentials('NDFC_SW_USERNAME')
        NDFC_SW_PASSWORD = credentials('NDFC_SW_PASSWORD')
        // WEBEX_TOKEN = credentials('WEBEX_TOKEN')
        // WEBEX_ROOM_ID = credentials('WEBEX_ROOM_ID')
        // DC_VXLAN_SCHEMA = "./nac-vxlan/schemas/schema.yaml"
        // DC_VXLAN_RULES = ""
        // ANSIBLE_HOST_KEY_CHECKING: 'false'
        // ANSIBLE_FORCE_COLOR: 'true'
        // ANSIBLE_COLLECTIONS_PATH: "./collections"
        // ANSIBLE_PERSISTENT_COMMAND_TIMEOUT: 1000
        // ANSIBLE_PERSISTENT_CONNECT_TIMEOUT: 1000
    }

    options {
        disableConcurrentBuilds()
    }

    stages {
        stage('Setup') {
            steps {
                sh 'rm -rf nac-vxlan'
                sh 'git clone --depth 1 --branch master https://wwwin-github.cisco.com/devegupt/nac-vxlan.git'
                // sh 'mkdir -p collections/ansible_collections/cisco'
                sh 'ansible-galaxy collection install -p collections/ansible_collections/ -r requirements.yaml -f'
                // sh 'rm -rf collections/ansible_collections/cisco/nac_dc_vxlan'
                // sh 'git clone --depth 1 --branch develop https://github.com/devegupt/ansible-dc-vxlan.git collections/ansible_collections/cisco/nac_dc_vxlan'
                }
            }
        stage('Validate') {
            steps {
                sh 'set -o pipefail && iac-validate host_vars/copy_netascode4_vrf_lite_ebgp -s nac-vxlan/schemas/schema.yaml -r collections/ansible_collections/cisco/nac_dc_vxlan/roles/validate/files/rules/ |& tee validate_output.txt'
            }
        }
        stage('Deploy') {
            steps {
                sh 'set -o pipefail && ansible-playbook -i inventory.yaml vxlan.yaml |& tee deploy_output.txt'
            }
        }
        stage('Test') {
            steps {
                sh 'set -o pipefail && iac-test -d host_vars/copy_netascode4_vrf_lite_ebgp -d nac-vxlan/defaults/defaults.yaml -f nac-vxlan/jinja_filters -t nac-vxlan/templates -o ./test_results |& tee test_output.txt'
                sh 'env'
            }
        // stage('cleanup') {
        //     steps {
        //         sh 'rm -rf nac-vxlan'
        //         sh 'rm -rf collections'
        //     }
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
    // BUILD_URL
    // JOB_URL
    // BUILD_NUMBER
    // JOB_BASE_NAME=ansible-dc-vxlan-testing
    // JOB_NAME=netascode/ansible-dc-vxlan-example/ansible-dc-vxlan-testing
    // }
}