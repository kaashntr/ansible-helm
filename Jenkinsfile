pipeline {
    agent any
    
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
                        sh 'sh tf apply --auto-approve -var-file=$SECRET_TFVARS_PATH'
                    }
                }
            }
        }
    }
}