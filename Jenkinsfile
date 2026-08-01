pipeline {
    agent any

    tools {
        nodejs 'NodeJS 25.2.1'
    }

    environment {
        IMAGE_NAME = "holumaintain/learn-jenkins-app:latest"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                bat 'npm install'
            }
        }

        stage('Run Tests') {
            steps {
                bat 'npm test -- --watchAll=false'
            }
        }

        stage('Build React App') {
            steps {
                bat 'npm run build'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'SonarScanner'

                    withSonarQubeEnv('SonarQube') {
                        bat """
                        "${scannerHome}\\bin\\sonar-scanner.bat" ^
                        -Dproject.settings=sonar-project.properties
                        """
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Docker Build') {
            steps {
                bat 'docker build -t %IMAGE_NAME% .'
            }
        }

        stage('Docker Push') {
            steps {
                bat 'docker push %IMAGE_NAME%'
            }
        }

        stage('Deploy Container') {
            steps {
                bat '''
                docker stop learn-jenkins-container 2>nul
                docker rm learn-jenkins-container 2>nul

                docker run -d ^
                  --name learn-jenkins-container ^
                  -p 3000:3000 ^
                  %IMAGE_NAME%
                '''
            }
        }
    }

    post {

        success {
            echo '==========================================='
            echo 'Pipeline completed successfully!'
            echo 'Code Tested'
            echo 'SonarQube Analysis Passed'
            echo 'Quality Gate Passed'
            echo 'Docker Image Built'
            echo 'Docker Image Pushed'
            echo 'Container Deployed'
            echo '==========================================='
        }

        failure {
            echo '==========================================='
            echo 'Pipeline Failed!'
            echo 'Check the Jenkins Console Output.'
            echo '==========================================='
        }

        always {
            cleanWs()
        }
    }
}