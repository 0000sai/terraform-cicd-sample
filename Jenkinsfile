pipeline {
    agent any

    parameters {
        choice(name: 'TF_VAR_environment', choices: ['dev', 'test', 'uat', 'prod'], description: 'Select Environment')
        choice(name: 'TERRAFORM_OPERATION', choices: ['plan', 'apply', 'destroy'], description: 'Select Terraform Operation')
    }

    environment {
        TF_VAR_environment = params.TF_VAR_environment
        AWS_ACCESS_KEY_ID = credentials('your-aws-access-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('your-aws-secret-access-key')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Terraform Init') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'your-aws-access-key-id', variable: 'AWS_ACCESS_KEY_ID'),
                                     string(credentialsId: 'your-aws-secret-access-key', variable: 'AWS_SECRET_ACCESS_KEY')]) {
                        sh 'terraform init'
                    }
                }
            }
        }

        stage('Terraform Workspace') {
            steps {
                script {
                    // Check if the Terraform workspace exists, create if not
                    def workspaceExists = sh(script: 'terraform workspace select ${TF_VAR_environment} || true', returnStatus: true) == 0

                    if (!workspaceExists) {
                        sh "terraform workspace new ${TF_VAR_environment}"
                    }
                }
            }
        }

        stage('Terraform Operation') {
            steps {
                script {
                    // Run Terraform based on the selected operation
                    switch(params.TERRAFORM_OPERATION) {
                        case 'plan':
                            sh "terraform plan -var='environment=${TF_VAR_environment}' -out=tfplan"
                            break
                        case 'apply':
                            sh 'terraform apply -auto-approve tfplan'
                            break
                        case 'destroy':
                            sh 'terraform destroy -auto-approve'
                            break
                        default:
                            error "Invalid Terraform operation selected"
                    }
                }
            }
        }
    }

    post {
        always {
            // Clean up artifacts, e.g., the Terraform plan file
            deleteDir()
        }
    }
}