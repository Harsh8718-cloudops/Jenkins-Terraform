pipeline {
    agent any

    parameters {
        booleanParam(name: 'autoApprove', defaultValue: false, description: 'Automatically apply?')
        choice(name: 'action', choices: ['apply', 'destroy'], description: 'Select Terraform action')
    }

    environment {
        AWS_ACCESS_KEY_ID     = credentials('access_key')
        AWS_SECRET_ACCESS_KEY = credentials('secret_access_key')
        AWS_DEFAULT_REGION    = 'us-east-1'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Harsh8718-cloudops/Jenkins-Terraform.git'
            }
        }

        stage('Terraform Init') {
            steps {
                sh 'terraform init'
            }
        }

        stage('Terraform Plan') {
            when {
                expression { params.action == 'apply' }
            }
            steps {
                sh 'terraform plan -out=tfplan'
                sh 'terraform show -no-color tfplan > tfplan.txt'
            }
        }

        stage('Apply / Destroy') {
            steps {
                script {
                    if (params.action == 'apply') {

                        if (!params.autoApprove) {
                            input message: "Approve Terraform Apply?",
                                  parameters: [
                                      text(name: 'Plan', defaultValue: readFile('tfplan.txt'))
                                  ]
                        }

                        sh "terraform apply -input=false tfplan"

                    } else if (params.action == 'destroy') {
                        sh 'terraform destroy --auto-approve'
                    }
                }
            }
        }
    }
}
