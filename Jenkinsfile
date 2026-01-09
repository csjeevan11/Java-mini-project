pipeline {
    agent { label 'agent1' }

    environment {
        JAVA_HOME = "/usr/lib/jvm/java-17-openjdk-amd64"
        PATH = "${JAVA_HOME}/bin:${PATH}"

        // SonarQube
        SONAR_HOST_URL = "http://<SONAR-IP>:9000"
        SONAR_PROJECT_KEY = "java-mini-project"

        // Artifactory
        ARTIFACTORY_URL = "http://<ARTIFACTORY-IP>:8081/artifactory"
        ART_REPO = "java-mini-project-local"

        // Tomcat
        TOMCAT_WEBAPPS = "/opt/tomcat/webapps"
    }

    stages {

        stage('Branch Validation') {
            when {
                expression { env.BRANCH_NAME == 'name.developer' }
            }
            steps {
                echo "Running pipeline for branch: ${BRANCH_NAME}"
            }
        }

        stage('Checkout Code') {
            when {
                expression { env.BRANCH_NAME == 'name.developer' }
            }
            steps {
                checkout scm
            }
        }

        stage('SonarQube Analysis') {
            when {
                expression { env.BRANCH_NAME == 'name.developer' }
            }
            steps {
                withSonarQubeEnv('SonarQube-Server') {
                    sh '''
                        mvn clean verify sonar:sonar \
                        -Dsonar.projectKey=${SONAR_PROJECT_KEY}
                    '''
                }
            }
        }

        stage('Quality Gate') {
            when {
                expression { env.BRANCH_NAME == 'name.developer' }
            }
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build WAR') {
            when {
                expression { env.BRANCH_NAME == 'name.developer' }
            }
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Upload WAR to Artifactory') {
            when {
                expression { env.BRANCH_NAME == 'name.developer' }
            }
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'jfrog-creds',
                    usernameVariable: 'ART_USER',
                    passwordVariable: 'ART_PASS'
                )]) {
                    sh '''
                        curl -u ${ART_USER}:${ART_PASS} \
                        -T target/*.war \
                        ${ARTIFACTORY_URL}/${ART_REPO}/
                    '''
                }
            }
        }

        stage('Deploy to Tomcat') {
            when {
                expression { env.BRANCH_NAME == 'name.developer' }
            }
            steps {
                sh '''
                    sudo cp target/*.war ${TOMCAT_WEBAPPS}/
                '''
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully"
        }
        failure {
            echo "Pipeline failed"
        }
    }
}
