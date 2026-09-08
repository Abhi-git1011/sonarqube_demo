pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
    }

    parameters {
        gitParameter(
            name: 'BRANCH_NAME',
            type: 'PT_BRANCH',
            branchFilter: 'origin/(.*)',
            defaultValue: 'main',
            selectedValue: 'DEFAULT',
            sortMode: 'DESCENDING_SMART',
            description: 'Select the GitHub branch to build'
        )
    }

    environment {
        GIT_URL = 'https://github.com/Abhi-git1011/sonarqube_demo.git'
        SONARQUBE_SERVER = 'SonarQube-Server'
    }

    stages {
        stage('Fetch Branch') {
            steps {
                echo "Fetching latest branch information from GitHub..."
                deleteDir()
                git branch: 'main', url: "${GIT_URL}"

                sh '''
                    echo "Fetching all branches..."
                    git fetch --all --prune

                    echo ""
                    echo "Available branches:"
                    git branch -r
                '''

                echo "Selected Branch: ${params.BRANCH_NAME}"
            }
        }

        stage('Checkout') {
            steps {
                echo "Checking out branch: ${params.BRANCH_NAME}"
                sh """
                    git checkout -B ${params.BRANCH_NAME} origin/${params.BRANCH_NAME}
                """
                sh '''
                    echo "========================================"
                    echo "Checked out branch:"
                    git branch --show-current

                    echo ""
                    echo "Commit:"
                    git rev-parse HEAD
                    echo "========================================"
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo "Starting SonarQube Analysis..."
                withSonarQubeEnv("${SONARQUBE_SERVER}") {
                    sh '''
                        mvn clean verify sonar:sonar \
                        -Dsonar.projectKey=devops-sonarqube-demo \
                        -Dsonar.projectName=devops-sonarqube-demo
                    '''
                }
                echo "SonarQube analysis completed."
            }
        }

        stage('Quality Gate') {
            steps {
                echo "Waiting for SonarQube Quality Gate..."
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
                echo "========================================"
                echo "CODE QUALITY GATE PASSED"
                echo "========================================"
            }
        }

        stage('Approval') {
            steps {
                script {
                    def approval = input(
                        message: 'Do you want to continue with the build?',
                        parameters: [
                            choice(name: 'APPROVAL', choices: ['Yes', 'No'])
                        ]
                    )
                    if (approval == 'No') {
                        error "Approval rejected. Pipeline failed."
                    }
                    echo "Approval granted."
                }
            }
        }

        stage('Build') {
            steps {
                dir('app') {
                    sh 'mvn package -DskipTests'
                }
            }
        }

        stage('Deploy to Nexus') {
            steps {
                dir('app') {
                    sh 'mvn deploy -DskipTests'
                }
            }
        }
    }
}
