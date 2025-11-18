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
        sh """
            echo "Deploying JAR to ${DEPLOY_DIR}"

            # Create directory
            sudo mkdir -p ${DEPLOY_DIR}

            # Copy jar from target folder
            sudo cp target/*.jar ${DEPLOY_DIR}/${JAR_NAME}

            # Stop existing app if running
            PID=\$(pgrep -f ${JAR_NAME}) || true
            if [ ! -z "\$PID" ]; then
                echo "Stopping existing app (PID: \$PID)"
                sudo kill -9 \$PID
            fi

            # Start new application
            echo "Starting application..."
            nohup java -jar ${DEPLOY_DIR}/${JAR_NAME} > ${DEPLOY_DIR}/app.log 2>&1 &
        """
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
