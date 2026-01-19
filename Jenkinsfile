pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    parameters {
        choice(
            name: 'action',
            choices: ['apply', 'destroy'],
            description: 'Terraform action to perform'
        )
        booleanParam(
            name: 'autoApprove',
            defaultValue: false,
            description: 'Skip manual approval before apply'
        )
    }

    environment {
        AWS_DEFAULT_REGION = 'us-east-1'
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
                withCredentials([
                    string(credentialsId: 'access_key', variable: 'AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'secret_access_key', variable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    sh '''
                      terraform --version
                      terraform init -input=false
                    '''
                }
            }
        }

        stage('Terraform Plan') {
            when {
                expression { params.action == 'apply' }
            }
            steps {
                withCredentials([
                    string(credentialsId: 'access_key', variable: 'AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'secret_access_key', variable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    sh '''
                      terraform plan -input=false -out=tfplan
                      terraform show -no-color tfplan > tfplan.txt
                    '''
                }
            }
        }

        stage('Approval') {
            when {
                allOf {
                    expression { params.action == 'apply' }
                    expression { !params.autoApprove }
                }
            }
            steps {
                input message: 'Approve Terraform Apply?',
                      parameters: [
                          text(
                              name: 'Terraform Plan',
                              defaultValue: readFile('tfplan.txt')
                          )
                      ]
            }
        }

        stage('Apply / Destroy') {
            steps {
                withCredentials([
                    string(credentialsId: 'access_key', variable: 'AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'secret_access_key', variable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    script {
                        if (params.action == 'apply') {
                            sh 'terraform apply -input=false tfplan'
                        } else {
                            sh 'terraform destroy -auto-approve -input=false'
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Terraform pipeline completed successfully ✅'
        }
        failure {
            echo 'Terraform pipeline failed ❌'
        }
        always {
            cleanWs()
        }
    }
}
