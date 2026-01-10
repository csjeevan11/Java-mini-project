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

        stage('Validate Branch') {
            when {
                branch 'name.developer'
            }
            steps {
                echo "Pipeline triggered for ${BRANCH_NAME}"
            }
        }

        stage('Checkout') {
            when {
                branch 'name.developer'
            }
            steps {
                checkout scm
            }
        }

        stage('SonarQube Scan') {
            when {
                branch 'name.developer'
            }
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
            when {
                branch 'name.developer'
            }
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build WAR') {
            when {
                branch 'name.developer'
            }
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Upload to Artifactory') {
            when {
                branch 'name.developer'
            }
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
