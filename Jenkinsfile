pipeline {
    agent any
    parameters {
        string(name: 'IMAGE_REPO', defaultValue: 'kaashntr/blood_is_fuel', description: 'Docker image repo')
        string(name: 'IMAGE_TAG', defaultValue: 'latest', description: 'Docker image tag')
        string(name: 'BACKEND_URL', defaultValue: 'https://back.class-schedule.pp.ua/', description: 'URL of backend for frontend')
        string(name: 'SERVICE_PORT', defaultValue: '3000', description: 'Service port to pod')
        string(name: 'RELEASE_STATE', defaultValue: 'present', description: 'Release state can be present or absent')
    }
    stages {
        stage('Checkout CSM') {
            steps {
                checkout scm
            }
        }
        stage('Create .ini locally'){
            steps {
                script {
                    if (fileExists('gcp-terraform/.git')) {
                        dir('gcp-terraform') {
                            sh 'git pull'
                        }
                    } else {
                        sh 'git clone https://github.com/kaashntr/gcp-terraform.git'
                    }
                }
                withCredentials([
                    file(credentialsId: 'gcp-sa-credentials', variable: 'SECRET_PATH'),
                    file(credentialsId: 'tfvars', variable: 'SECRET_TFVARS_PATH'),
                ]){
                    dir ('gcp-terraform'){
                        sh '''
                            export GOOGLE_APPLICATION_CREDENTIALS=$SECRET_PATH
                            terraform init
                            terraform apply -var-file="$SECRET_TFVARS_PATH" -auto-approve
                        '''
                    }
                }
                script {
                    dir('ansible'){
                        ansiblePlaybook(
                            playbook: 'install_front.yml',
                            inventory: 'inventory.ini',
                            colorized: true
                            extraVars: [
                                image_repository: params.IMAGE_REPO,
                                image_tag: params.IMAGE_TAG,
                                service_port: params.SERVICE_PORT,
                                BACKEND_URL: params.BACKEND_URL,
                                helm_release_state: params.RELEASE_STATE
                            ]
                        )
                    }
                }
            }
        }
    }
}