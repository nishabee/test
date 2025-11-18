pipeline {
    agent any

    environment {
        // Jenkins Credential IDs (configured in Jenkins credentials)
        SONAR_TOKEN = credentials('SONAR_TOKEN')   // secret text
        // Optionally, GitHub PAT if needed: GITHUB_PAT = credentials('GITHUB_PAT')
        DEPLOY_DIR = '/opt/springapp'
        APP_JAR = 'app.jar'
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout code from repo (Jenkins job should be already linked; this ensures latest)
                git branch: 'main', url: 'https://github.com/nishabee/test.git'
            }
        }

        stage('Build & Test') {
            steps {
                // Run maven build and tests
                sh 'mvn -B -e -V clean verify'
            }
        }

        stage('SonarCloud Analysis') {
            steps {
                // Run SonarCloud analysis. Ensure sonar.organization and sonar.projectKey are filled if required by SonarCloud
                // Replace sonar.organization and sonar.projectKey with your actual values if SonarCloud asks for them.
                sh '''
                   mvn -B sonar:sonar \
                     -Dsonar.login=${SONAR_TOKEN} \
                     -Dsonar.organization="nishabee" \
                     -Dsonar.projectKey="nishabee_test" \
                     || { echo "Sonar analysis failed"; exit 1; }
                '''
            }
        }

        stage('Trivy Vulnerability Scan') {
            steps {
                // Scan the project directory (source + dependencies). Exit with code 1 if vulnerabilities found.
                // Trivy will fetch vulnerability DB (first run may download databases)
                sh '''
                   echo "Running Trivy filesystem scan (this may take a while on first run)..."
                   trivy fs --security-checks vuln --exit-code 1 --format table --no-progress .
                '''
            }
        }

        stage('Package') {
            steps {
                sh '''
                  # copy the built jar to app.jar in workspace root
                  cp target/*.jar ${APP_JAR}
                  ls -lh ${APP_JAR} || true
                '''
            }
        }

        stage('Deploy to EC2 (same server)') {
            steps {
                // Copy jar to /opt/springapp and restart systemd
                sh '''
                  sudo cp ${APP_JAR} ${DEPLOY_DIR}/${APP_JAR}
                  # ensure perms allow jenkins user to read the jar
                  sudo chown jenkins:jenkins ${DEPLOY_DIR}/${APP_JAR}
                  sudo systemctl restart springapp
                  sudo systemctl status --no-pager springapp || true
                '''
            }
        }
    }

    post {
        success {
            echo "Pipeline completed SUCCESS"
        }
        failure {
            echo "Pipeline FAILED"
        }
    }
}
