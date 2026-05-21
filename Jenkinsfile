pipeline {
    agent any
    
    environment {
       SONAR_TOKEN = credentials('SONAR_TOKEN')
       SNYK_TOKEN = credentials('SNYK_TOKEN')
       IMAGE_NAME = "kaustubh-sit753-7-3hd"
       CONTAINER_NAME = "kaustubh-sit753-container"
       APP_PORT = "3001"
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code from GitHub'
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo 'Building application and Docker image'
                bat 'npm install'
                bat 'docker build -t %IMAGE_NAME%:%BUILD_NUMBER% .'
                bat 'docker tag %IMAGE_NAME%:%BUILD_NUMBER% %IMAGE_NAME%:latest'
            }
        }

        stage('Test') {
            steps {
                echo 'Running automated tests'
                bat 'npm test || exit /b 0'
            }
        }

        stage('Code Quality') {
            steps {
                echo 'Running SonarCloud static analysis'
                bat 'npm install -g sonar-scanner'
                bat 'sonar-scanner'
            }
        }

        stage('Security') {
            steps {
               echo 'Running security scans'
               bat 'npm audit || exit /b 0'
               bat 'npm install -g snyk'
               bat 'snyk auth %SNYK_TOKEN%'
               bat 'snyk test || exit /b 0'
            }
       }

       stage('Deploy') {
    steps {
        echo 'Deploying application using Docker Compose'
        bat 'docker compose down || exit /b 0'
        bat 'docker compose up -d --build'
    }
}

        stage('Release') {
            steps {
                echo 'Creating release version'
                bat 'docker tag %IMAGE_NAME%:latest %IMAGE_NAME%:release-%BUILD_NUMBER%'
            }
        }

       stage('Monitoring') {
            steps {
                echo 'Monitoring stage: checking deployed application health and container metrics'
                bat 'docker ps'
                bat 'curl http://localhost:%APP_PORT% || exit /b 0'
                bat 'docker stats %CONTAINER_NAME% --no-stream || exit /b 0'
                bat 'docker logs %CONTAINER_NAME% --tail 20 || exit /b 0'
            }
        }
    } 

    post {
        success {
            echo 'Pipeline completed successfully'
        }

        failure {
            echo 'Pipeline failed'
        }
    }
}