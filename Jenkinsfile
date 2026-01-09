pipeline {
    agent { label 'agent1' }

    environment {
        // Java
        JAVA_HOME = "/usr/lib/jvm/java-17-openjdk-amd64"

        // Maven
        MAVEN_HOME = "/opt/maven"

        // PATH
        PATH = "${JAVA_HOME}/bin:${MAVEN_HOME}/bin:${env.PATH}"

        // SonarQube URL
        SONARQUBE_URL = "http://3.110.219.224:9000"

        // Artifactory
        ARTIFACTORY_URL = "http://172.31.40.213:8081/artifactory"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/csjeevan11/Java-mini-project.git',
                    credentialsId: 'github'
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
                    withSonarQubeEnv('sonar-server') {
                        withCredentials([string(credentialsId: 'Sonar-token', variable: 'SONAR_TOKEN')]) {
                            sh '''
                                mvn clean verify sonar:sonar \
                                -Dsonar.projectKey=JavaMiniProject \
                                -Dsonar.host.url=${SONARQUBE_URL} \
                                -Dsonar.login=${SONAR_TOKEN}
                            '''
                        }
                    }
                }
            }
        }

        stage('Upload to Artifactory') {
            steps {
                dir('sample-app/target') {
                    withCredentials([usernamePassword(
                        credentialsId: 'jfrog-creds',
                        usernameVariable: 'ART_USER',
                        passwordVariable: 'ART_PASS'
                    )]) {
                        sh '''
                            curl -u ${ART_USER}:${ART_PASS} \
                            -T *.war \
                            ${ARTIFACTORY_URL}/java-mini-project-local/
                        '''
                    }
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                dir('sample-app/target') {
                    sh '''
                        sudo cp *.war /opt/tomcat/webapps/
                        sudo systemctl restart tomcat10
                    '''
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
