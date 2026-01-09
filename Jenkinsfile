pipeline {
    agent { label 'agent1' }

    environment {
        // Java
        JAVA_HOME = "/usr/lib/jvm/java-17-openjdk-amd64"
        PATH = "${JAVA_HOME}/bin:${PATH}"

        // SonarQube
        SONARQUBE_URL = "http://3.110.219.224:9000"
        SONARQUBE_AUTH_TOKEN = credentials('Sonar-token')

        // Artifactory
        ARTIFACTORY_URL = "http://172.31.40.213:8081/artifactory"
        ARTIFACTORY_CREDS = credentials('jfrog-creds')
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/csjeevan11/Java-mini-project.git'
            }
        }

        stage('Build') {
            steps {
                dir('sample-app') {
                    sh 'mvn clean package -DskipTests'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                dir('sample-app') {
                    sh """
                    mvn sonar:sonar \
                    -Dsonar.projectKey=JavaMiniProject \
                    -Dsonar.host.url=${SONARQUBE_URL} \
                    -Dsonar.login=${SONARQUBE_AUTH_TOKEN}
                    """
                }
            }
        }

        stage('Upload to Artifactory') {
            steps {
                dir('sample-app/target') {
                    sh """
                    curl -u ${ARTIFACTORY_CREDS_USR}:${ARTIFACTORY_CREDS_PSW} \
                    -T *.war \
                    ${ARTIFACTORY_URL}/java-mini-project-local/
                    """
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                dir('sample-app/target') {
                    sh """
                    sudo cp *.war /opt/tomcat/webapps/
                    sudo systemctl restart tomcat10
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline executed successfully on agent1"
        }
        failure {
            echo "Pipeline failed. Check logs."
        }
    }
}
