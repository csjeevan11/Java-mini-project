pipeline {
    agent any

    environment {
        JAVA_HOME  = "/usr/lib/jvm/java-17-openjdk-amd64"
        MAVEN_HOME = "/opt/maven"
        PATH = "${JAVA_HOME}/bin:${MAVEN_HOME}/bin:${env.PATH}"

        SONARQUBE_URL = "http://3.110.46.96:9000"
        ARTIFACTORY_REPO_URL = "http://3.110.46.96:8081/artifactory/java-mini-project-local"
    }

    triggers {
        githubPush()
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('SonarQube Analysis') {
            steps {
                dir('sample-app') {
                    withSonarQubeEnv('sonar-server') {
                        sh '''
                            mvn clean verify sonar:sonar \
                            -Dsonar.projectKey=JavaMiniProject
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

        stage('Build & Deploy to Artifactory') {
            steps {
                dir('sample-app') {
                    withCredentials([
                        usernamePassword(
                            credentialsId: 'jfrog-creds',
                            usernameVariable: 'ART_USER',
                            passwordVariable: 'ART_PASS'
                        )
                    ]) {
                        sh '''
                            mvn clean deploy -DskipTests \
                            -DaltDeploymentRepository=artifactory::default::${ARTIFACTORY_REPO_URL} \
                            -Dusername=${ART_USER} \
                            -Dpassword=${ART_PASS}
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
                        sudo systemctl restart tomcat
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
            echo "Pipeline failed (Sonar / Build / Deploy error)"
        }
    }
}
