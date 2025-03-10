pipeline {
    parameters {
        booleanParam(name: 'autoApprove', defaultValue: false, description: 'Automatically run apply after generating plan?')
    }

    environment {
        AWS_ACCESS_KEY_ID = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
    }

    agent any

    stages {
        stage('Checkout') {
            steps {
                script {
                    dir("terraform") {
                        git "https://github.com/sagarkulkarni1989/Terraform-Jenkins.git"
                    }
                }
            }
        }

        stage('Plan') {
            steps {
                sh 'cd terraform/ ; terraform init'
                sh 'cd terraform/ ; terraform plan -out tfplan'
                sh 'cd terraform/ ; terraform show -no-color tfplan > tfplan.txt'
            }
        }

        stage('Approval') {
            when {
                not {
                    equals expected: true, actual: params.autoApprove
                }
            }

            steps {
                script {
                    def plan = readFile 'terraform/tfplan.txt'
                    timeout(time: 5, unit: 'DAYS') {
                        input message: "Do you want to apply the plan?",
                              parameters: [text(name: 'Plan', description: 'Please review the plan', defaultValue: plan)]
                    }
                }
            }
        }

        stage('Apply') {
            steps {
                sh 'cd terraform/ ; terraform apply -input=false tfplan'
            }
        }
    }
}
