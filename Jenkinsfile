pipeline {
    agent any

    triggers {
        githubPush() // Triggers build on every Git push
    }

    environment {
        SERVER_USER = 'tomcat'
        SERVER_HOST = '3.15.21.212'
        SERVER_PATH = '/opt/tomcat/webapps/' 
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/ideastarboy/NumberGuessGame.git'
            }
        }

        stage('Code Quality Check') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn clean verify sonar:sonar'
                }
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests' // Skip tests in build stage
            }
        }

        stage('Run Tests') {
            steps {
                sh 'mvn test' // Run JUnit tests
            }
        }

        stage('Deploy to Server') {
            steps {
                sshagent(['ad94a7ca-9f84-47c5-bcb3-89d95070954c']) { 
                    sh '''
                    scp target/*.war ${SERVER_USER}@${SERVER_HOST}:${SERVER_PATH}
                    ssh ${SERVER_USER}@${SERVER_HOST} "sudo systemctl restart tomcat.service"
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment successful!'
        }
        failure {
            mail to: 'adewoleife@gmail.com',
                 subject: "Build Failed: ${currentBuild.fullDisplayName}",
                 body: "Check Jenkins logs for details."
        }
    }
}
