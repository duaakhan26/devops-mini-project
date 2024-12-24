pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                script {
                    // Use SSH to pull from your GitHub repository
                    checkout([$class: 'GitSCM',
                        branches: [[name: "*/${env.BRANCH_NAME}"]],
                        userRemoteConfigs: [[
                            url: 'git@github.com:duaakhan26/devops-mini-project.git',
                            credentialsId: 'GITHUB_KEY_SSH' // Updated to the correct Jenkins SSH credentials
                        ]]
                    ])
                }
            }
        }

        stage('Build Docker Image') {
            when {
                anyOf {
                    branch 'dev'
                    branch 'testing'
                    branch 'main'
                }
            }
            steps {
                script {
                    echo "Building Docker image for branch: ${env.BRANCH_NAME}"
                    sh """
                      docker build -t my-node-app:${env.BRANCH_NAME} .
                    """
                }
            }
        }

        stage('Deploy to Dev') {
            when {
                branch 'dev'
            }
            steps {
                script {
                    echo "Deploying to Dev environment..."
                    sshagent (credentials: ['DEV_KEY_SSH']) {
                        sh """
                          ssh -i "devops-mini-project.pem" -o StrictHostKeyChecking=no ubuntu@ec2-16-171-111-82.eu-north-1.compute.amazonaws.com \\
                          'docker stop app_dev || true && docker rm app_dev || true && \\
                           docker run -d --name app_dev -p 3000:3000 my-node-app:dev'
                        """
                    }
                }
            }
        }

        stage('Deploy to Testing') {
            when {
                branch 'testing'
            }
            steps {
                script {
                    echo "Deploying to Testing environment..."
                    sshagent (credentials: ['DEV_KEY_SSH']) {
                        sh """
                          ssh -i "devops-mini-project.pem" -o StrictHostKeyChecking=no ubuntu@ec2-51-20-192-98.eu-north-1.compute.amazonaws.com \\
                          'docker stop app_testing || true && docker rm app_testing || true && \\
                           docker run -d --name app_testing -p 3000:3000 my-node-app:testing'
                        """
                    }
                }
            }
        }

        stage('Run Tests in Testing Environment') {
            when {
                branch 'testing'
            }
            steps {
                script {
                    echo "Running automated tests on the Testing environment..."
                    sshagent (credentials: ['DEV_KEY_SSH']) {
                        sh """
                          ssh -i "devops-mini-project.pem" -o StrictHostKeyChecking=no ubuntu@ec2-51-20-192-98.eu-north-1.compute.amazonaws.com \\
                          'docker exec app_testing npm test'
                        """
                    }
                }
            }
        }

        stage('Merge Testing to Main') {
            when {
                allOf {
                    branch 'testing'
                    expression { currentBuild.currentResult == "SUCCESS" }
                }
            }
            steps {
                script {
                    echo "All tests passed. Merging from testing to main..."
                    // Uncomment if automatic merging is needed
                    // sshagent (credentials: ['GITHUB_KEY_SSH']) {
                    //     sh """
                    //       git config user.name 'duaakhan26'
                    //       git config user.email 'your_email@example.com'
                    //       git checkout main
                    //       git merge origin/testing
                    //       git push origin main
                    //     """
                    // }
                }
            }
        }

        stage('Deploy to Staging') {
            when {
                anyOf {
                    branch 'main'
                    branch 'master'
                }
            }
            steps {
                script {
                    echo "Deploying to Staging environment..."
                    sshagent (credentials: ['DEV_KEY_SSH']) {
                        sh """
                          ssh -i "devops-mini-project.pem" -o StrictHostKeyChecking=no ubuntu@ec2-13-61-151-244.eu-north-1.compute.amazonaws.com \\
                          'docker stop app_staging || true && docker rm app_staging || true && \\
                           docker run -d --name app_staging -p 3000:3000 my-node-app:main'
                        """
                    }
                }
            }
        }
    }
}
