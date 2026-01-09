pipeline {
    agent { label 'agent1' }

    environment {
        JAVA_HOME  = "/usr/lib/jvm/java-17-openjdk-amd64"
        MAVEN_HOME = "/opt/maven"
        PATH = "${JAVA_HOME}/bin:${MAVEN_HOME}/bin:${env.PATH}"
        SONARQUBE_URL = "http://3.110.219.224:9000"
        ARTIFACTORY_REPO_URL = "http://172.31.40.213:8081/artifactory/java-mini-project-local"
    }

    triggers {
        githubPush()
    }

    stages {

        stage('Branch Validation') {
            when {
                expression {
                    env.BRANCH_NAME == 'name.developer'
                }
            }
            steps {
                echo "Triggered for branch: ${env.BRANCH_NAME}"
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

        stage('SonarQube Analysis') {
            when {
                branch 'name.developer'
            }
            steps {
                dir('sample-app') {
                    withSonarQubeEnv('sonar-server') {
                        withCredentials([
                            string(credentialsId: 'Sonar-token', variable: 'SONAR_TOKEN')
                        ]) {
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

        stage('Build & Deploy to Artifactory') {
            when {
                branch 'name.developer'
            }
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
            when {
                branch 'name.developer'
            }
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
