pipeline {
    agent { label 'agent1' }

    environment {
        JAVA_HOME = "/usr/lib/jvm/java-17-openjdk-amd64"
        PATH = "${JAVA_HOME}/bin:${PATH}"

        SONAR_PROJECT_KEY = "java-mini-project"

        ARTIFACTORY_URL = "http://<ARTIFACTORY-IP>:8081/artifactory"
        ART_REPO = "java-mini-project-local"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('SonarQube Scan') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh """
                      mvn clean verify sonar:sonar \
                      -Dsonar.projectKey=${SONAR_PROJECT_KEY}
                    """
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

        stage('Build WAR') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Upload to Artifactory') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'jfrog-creds',
                    usernameVariable: 'ART_USER',
                    passwordVariable: 'ART_PASS'
                )]) {
                    sh """
                      curl -u ${ART_USER}:${ART_PASS} \
                      -T target/*.war \
                      ${ARTIFACTORY_URL}/${ART_REPO}/
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully"
        }
        failure {
            echo "Pipeline failed "
        }
    }
}
