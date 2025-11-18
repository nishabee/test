pipeline {

    agent any

    environment {
        // Maven Home (if installed manually)
        PATH = "/usr/share/maven/bin:$PATH"

        // SonarCloud
        SONAR_TOKEN = credentials('SONAR_TOKEN')
        SONAR_ORG = "nishabee"
        SONAR_PROJECT = "nishabee_test"

        // Deployment
        DEPLOY_DIR = "/home/ubuntu/app"
        JAR_NAME = "demo-1.0.jar"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/nishabee/test.git'
            }
        }

        stage('Build & Unit Test') {
            steps {
                sh 'mvn clean install -DskipTests=false'
            }
        }

        stage('SonarCloud Analysis') {
            steps {
                sh """
                    mvn sonar:sonar \
                        -Dsonar.host.url=https://sonarcloud.io \
                        -Dsonar.organization=${SONAR_ORG} \
                        -Dsonar.projectKey=${SONAR_PROJECT} \
                        -Dsonar.login=${SONAR_TOKEN}
                """
            }
        }

        stage('Trivy Vulnerability Scan') {
            steps {
                sh '''
                    trivy fs --exit-code 0 --severity HIGH,CRITICAL .
                '''
            }
        }

        stage('Package JAR') {
            steps {
                sh 'mvn -DskipTests package'
            }
        }

        stage('Deploy to EC2 (same server)') {
            steps {
                sh '''
                    DEPLOY_DIR=''' + "${DEPLOY_DIR}" + ''''
                    JAR_NAME=''' + "${JAR_NAME}" + ''''

                    echo "Deploying JAR to $DEPLOY_DIR"

                    sudo mkdir -p $DEPLOY_DIR
                    sudo cp target/*.jar $DEPLOY_DIR/$JAR_NAME

                    echo "Checking for running JAR process..."

                    PID=$(pgrep -f $JAR_NAME) || true
                    if [ ! -z "$PID" ]; then
                        echo "Killing existing process: $PID"
                        sudo kill -9 $PID
                    fi

                    echo "Starting new application..."
                    nohup java -jar $DEPLOY_DIR/$JAR_NAME > $DEPLOY_DIR/app.log 2>&1 &
                '''
            }
        }
    }

    post {
        success {
            echo "Pipeline SUCCESS"
        }
        failure {
            echo "Pipeline FAILED"
        }
    }
}
