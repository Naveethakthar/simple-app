pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "naveethakthar/simple-app"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Naveethakthar/simple-app.git'
            }
        }

        stage('Build & Test') {
            steps {
                bat '''
                pip install pytest
                pytest
                '''
            }
        }
stage('SonarQube Analysis') {
    steps {
        script {
            def scannerHome = tool 'sonar-scanner'
            withSonarQubeEnv('sonar') {
                bat """
                ${scannerHome}\\bin\\sonar-scanner ^
                -Dsonar.projectKey=simple-app ^
                -Dsonar.sources=.
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
                bat "docker build -t %DOCKER_IMAGE%:latest ."
            }
        }

       stage('Push to DockerHub') {
    environment {
        DOCKER_IMAGE = "naveethakthar/simple-app"
    }
    steps {
        // ← THIS LINE BUILDS THE DOCKER IMAGE
        bat "docker build -t %DOCKER_IMAGE%:latest ."

        // Login and push
        withCredentials([usernamePassword(
            credentialsId: 'dockerhub-creds',
            usernameVariable: 'USER',
            passwordVariable: 'PASS'
        )]) {
            bat ''' 
            echo %PASS% | docker login -u %USER% --password-stdin
            docker push %DOCKER_IMAGE%:latest
            '''
        }
    }
}


        stage('Deploy') {
            steps {
                bat '''
                docker stop simple-app || exit 0
                docker rm simple-app || exit 0
                docker run -d --name simple-app %DOCKER_IMAGE%:latest
                '''
            }
        }
    }
}
