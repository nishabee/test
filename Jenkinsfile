pipeline {
    agent any

    stages {

        stage('Pull from GitHub') {
            steps {
                git branch: 'main', url: 'https://github.com/nishabee/test.git'
            }
        }

        stage('Build & Test') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('Create .jar') {
            steps {
                sh 'cp target/*.jar app.jar'
            }
        }

        stage('Deploy to EC2 (Same Server)') {
            steps {
                sh '''
                cp app.jar /opt/springapp/app.jar
                sudo systemctl restart springapp
                '''
            }
        }
    }
}

