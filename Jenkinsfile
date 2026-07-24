pipeline {
    agent any

    parameters {
        booleanParam(name: 'autoApprove', defaultValue: false, description: 'Automatically run apply after generating plan?')
        choice(name: 'action', choices: ['apply', 'destroy'], description: 'Select the action to perform')
    }

    environment {
        AWS_ACCESS_KEY_ID     = credentials('aws-access-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-access-key')
        AWS_DEFAULT_REGION    = 'us-east-1'
    }

    stages {
        stage('Checkout') {
            steps {
                echo '========== Checking out code from repository =========='
                git branch: 'main', url: 'https://github.com/shekahassan/Terraform-Jenkins-Infra-Pipeline.git'
                echo '✓ Code checkout completed successfully'
            }
        }

        stage('Terraform Initialize') {
            steps {
                echo '========== Initializing Terraform working directory =========='
                sh '''
                    echo "Terraform version:"
                    terraform version
                    echo ""
                    echo "Running terraform init..."
                    terraform init -upgrade
                '''
                echo '✓ Terraform initialization completed successfully'
            }
        }

        stage('Terraform Validate') {
            steps {
                echo '========== Validating Terraform configuration =========='
                sh '''
                    echo "Validating Terraform configuration..."
                    terraform validate
                    echo ""
                    echo "Formatting check..."
                    terraform fmt -check -recursive . || echo "Warning: Some files need formatting"
                '''
                echo '✓ Configuration validation completed'
            }
        }

        stage('Terraform Format') {
            steps {
                echo '========== Formatting Terraform files =========='
                sh '''
                    echo "Applying Terraform format..."
                    terraform fmt -recursive .
                    echo "Format applied successfully"
                '''
                echo '✓ Terraform formatting completed'
            }
        }

        stage('Terraform Plan') {
            steps {
                echo '========== Generating Terraform execution plan =========='
                sh '''
                    echo "Running terraform plan..."
                    terraform plan -out=tfplan
                    echo ""
                    echo "Detailed plan output:"
                    terraform show -no-color tfplan > tfplan.txt
                    cat tfplan.txt
                '''
                echo '✓ Terraform plan generated successfully'
            }
        }

        stage('Manual Review and Approval') {
            when {
                expression {
                    return !params.autoApprove
                }
            }
            steps {
                echo '========== Awaiting manual approval =========='
                script {
                    def plan = readFile 'tfplan.txt'
                    input message: '⚠️  Please review the Terraform plan above ⚠️',
                    parameters: [text(name: 'Plan', description: 'Review the plan carefully before proceeding:', defaultValue: plan)]
                }
                echo '✓ Plan approved by user'
            }
        }

        stage('Terraform Apply') {
            when {
                expression {
                    return params.action == 'apply'
                }
            }
            steps {
                echo '========== Applying Terraform configuration =========='
                sh '''
                    echo "Applying terraform plan..."
                    terraform apply -input=false tfplan
                    echo ""
                    echo "Fetching output values..."
                    terraform output
                '''
                echo '✓ Terraform apply completed successfully'
            }
        }

        stage('Terraform Destroy') {
            when {
                expression {
                    return params.action == 'destroy'
                }
            }
            steps {
                echo '========== Destroying Terraform resources =========='
                sh '''
                    echo "⚠️  WARNING: About to destroy infrastructure ⚠️"
                    echo "Destroying all resources managed by Terraform..."
                    terraform destroy --auto-approve
                '''
                echo '✓ Terraform destroy completed successfully'
            }
        }

        stage('Output Results') {
            when {
                expression {
                    return params.action == 'apply'
                }
            }
            steps {
                echo '========== Displaying Infrastructure Outputs =========='
                sh '''
                    echo "Infrastructure deployed successfully!"
                    echo ""
                    echo "Output values:"
                    terraform output -json
                '''
                echo '✓ All outputs displayed'
            }
        }
    }
}