pipeline {
    agent any

    environment {
        JAVA_HOME = "/usr/lib/jvm/java-17-openjdk-amd64"
        PATH = "${JAVA_HOME}/bin:${env.PATH}"

        SONAR_PROJECT_KEY = "java-mini-project"
        SONARQUBE_URL = "http://3.110.46.96:9000"

        ARTIFACTORY_REPO_URL = "http://3.110.46.96:8081/artifactory/java-mini-project-local"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    withCredentials([
                        string(credentialsId: 'Sonar-token', variable: 'SONAR_TOKEN')
                    ]) {
                        sh '''
                            mvn clean verify sonar:sonar \
                            -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                            -Dsonar.host.url=${SONARQUBE_URL} \
                            -Dsonar.login=${SONAR_TOKEN}
                        '''
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

        stage('Build WAR') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Upload to Artifactory') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'jfrog-creds',
                        usernameVariable: 'ART_USER',
                        passwordVariable: 'ART_PASS'
                    )
                ]) {
                    sh '''
                        curl -u ${ART_USER}:${ART_PASS} \
                        -T target/*.war \
                        ${ARTIFACTORY_REPO_URL}/
                    '''
                }
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
