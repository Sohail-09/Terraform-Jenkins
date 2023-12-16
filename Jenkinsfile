pipeline {
    parameters {
        booleanParam(name: 'autoApprove', defaultValue: false, description: 'Automatically run apply after generating plan?')
        booleanParam(name: 'destroyResources', defaultValue: false, description: 'Destroy resources if no resources to create?')
    }

    environment {
        AWS_ACCESS_KEY_ID     = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
    }

    agent any

    stages {
        stage('checkout') {
            steps {
                script {
                    dir("terraform") {
                        git "https://github.com/yeshwanthlm/Terraform-Jenkins.git"
                    }
                }
            }
        }

        stage('Plan') {
            steps {
                script {
                    bat(script: 'cd terraform && terraform init', label: 'Terraform Init')
                    bat(script: 'cd terraform && terraform plan -out tfplan', label: 'Terraform Plan')
                    bat(script: 'cd terraform && terraform show -no-color tfplan > tfplan.txt', label: 'Terraform Show')

                    def planOutput = readFile 'terraform/tfplan.txt'
                    if (planOutput.contains('No changes')) {
                        echo 'No resources to create. Skipping apply stage.'
                        currentBuild.result = 'SUCCESS'  // Mark the build as successful to skip subsequent stages
                    }
                }
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
                    input message: "Do you want to apply the plan?",
                          parameters: [text(name: 'Plan', description: 'Please review the plan', defaultValue: plan)]
                }
            }
        }

        // stage('Apply') {
        //     when {
        //         expression { params.autoApprove && currentBuild.resultIsBetterOrEqualTo('SUCCESS') }
        //     }
        //     steps {
        //         script {
        //             bat(script: 'cd terraform && terraform apply -input=false tfplan', label: 'Terraform Apply')
        //         }
        //     }
        // }

        stage('Destroy') {
            when {
                allOf {
                    expression { params.destroyResources }
                    expression { currentBuild.resultIsBetterOrEqualTo('SUCCESS') }
                }
            }
            steps {
                script {
                    def userInput = input message: 'Do you want to destroy resources?',
                                         parameters: [booleanParam(defaultValue: false, description: 'Proceed with destruction?')]
                    if (userInput) {
                        bat(script: 'cd terraform && terraform destroy -auto-approve', label: 'Terraform Destroy')
                    } else {
                        echo 'Destruction of resources canceled.'
                    }
                }
            }
        }
    }
}


// pipeline {

//     parameters {
//         booleanParam(name: 'autoApprove', defaultValue: false, description: 'Automatically run apply after generating plan?')
//     } 
//     environment {
//         AWS_ACCESS_KEY_ID     = credentials('AWS_ACCESS_KEY_ID')
//         AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
//     }

//    agent  any
//     stages {
//         stage('checkout') {
//             steps {
//                  script{
//                         dir("terraform")
//                         {
//                             git "https://github.com/yeshwanthlm/Terraform-Jenkins.git"
//                         }
//                     }
//                 }
//             }

//         stage('Plan') {
//             steps {
//                 sh 'pwd;cd terraform/ ; terraform init'
//                 sh "pwd;cd terraform/ ; terraform plan -out tfplan"
//                 sh 'pwd;cd terraform/ ; terraform show -no-color tfplan > tfplan.txt'
//             }
//         }
//         stage('Approval') {
//            when {
//                not {
//                    equals expected: true, actual: params.autoApprove
//                }
//            }

//            steps {
//                script {
//                     def plan = readFile 'terraform/tfplan.txt'
//                     input message: "Do you want to apply the plan?",
//                     parameters: [text(name: 'Plan', description: 'Please review the plan', defaultValue: plan)]
//                }
//            }
//        }

//         stage('Apply') {
//             steps {
//                 sh "pwd;cd terraform/ ; terraform apply -input=false tfplan"
//             }
//         }
//     }

//   }
