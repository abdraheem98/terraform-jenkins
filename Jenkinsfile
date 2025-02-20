pipeline {
    agent any

    parameters {
        booleanParam(name: 'autoApprove', defaultValue: false, description: 'Automatically run apply after generating plan?')
        choice(name: 'action', choices: ['apply', 'destroy'], description: 'Select the action to perform')
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/abdraheem98/terraform-jenkins.git'
            }
        }
        stage('Terraform init') {
            steps {
                withCredentials([
                    string(credentialsId: 'aws-access-key-id', variable: 'AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'aws-secret-access-key', variable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    sh 'terraform init'
                }
            }
        }
        stage('Plan') {
            steps {
                withCredentials([
                    string(credentialsId: 'aws-access-key-id', variable: 'AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'aws-secret-access-key', variable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    sh 'terraform plan -out tfplan'
                    sh 'terraform show -no-color tfplan > tfplan.txt'
                }
            }
        }
        stage('Apply / Destroy') {
            steps {
                script {
                    if (params.action == 'apply') {
                        if (!params.autoApprove) {
                            def plan = readFile 'tfplan.txt'
                            input message: "Do you want to apply the plan?",
                            parameters: [text(name: 'Plan', description: 'Please review the plan', defaultValue: plan)]
                        }

                        withCredentials([
                            string(credentialsId: 'aws-access-key-id', variable: 'AWS_ACCESS_KEY_ID'),
                            string(credentialsId: 'aws-secret-access-key', variable: 'AWS_SECRET_ACCESS_KEY')
                        ]) {
                            sh "terraform apply -input=false tfplan"
                        }
                    } else if (params.action == 'destroy') {
                        withCredentials([
                            string(credentialsId: 'aws-access-key-id', variable: 'AWS_ACCESS_KEY_ID'),
                            string(credentialsId: 'aws-secret-access-key', variable: 'AWS_SECRET_ACCESS_KEY')
                        ]) {
                            sh "terraform destroy --auto-approve"
                        }
                    } else {
                        error "Invalid action selected. Please choose either 'apply' or 'destroy'."
                    }
                }
            }
        }
    }
}
