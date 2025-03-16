pipeline {
    options { timestamps () }  // time stamps for easier log tracking 
    
    agent any

    triggers {
        githubPush() // Triggers build on every Git push
    }

    environment {
        SERVER_HOST = '18.117.75.126'
        TOMCAT_PATH = '/opt/tomcat/webapps/' 
        SONAR_TOKEN = credentials('sonar-new')
        WAR_FILE_PATH = '/var/lib/jenkins/workspace/Project/target/NumberGuessGame-1.0-SNAPSHOT.war'
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/ideastarboy/NumberGuessGame.git'
            }
        }

        stage('Code Quality Check') {
            steps {
                withSonarQubeEnv('Sonar') {
                    sh 'mvn clean verify sonar:sonar -Dsonar.login=$SONAR_TOKEN'
                }
            }
        }

        stage('Build') {
            tools {
                maven 'Maven'
                
            }
            steps {
                sh 'mvn clean package -X' // Skip tests in build stage -X shows full debug logs
            }
        }

        stage('Run Tests') {
            steps {
                sh 'mvn test' // Run JUnit tests
            }
            
        }

        stage('Deploy to Server') {
            steps {
                script {
                    sh '''
                    sudo cp ${WAR_FILE_PATH} ${TOMCAT_PATH}
                    '''
                    sh '''
                    sudo systemctl restart tomcat
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
