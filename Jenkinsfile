pipeline {
    agent { label 'agent2' }

    tools {
        maven 'Maven-3.8.7'
        jdk 'OpenJDK-21.0.9'
    }

    stages {

        stage('Build') {
            steps {
                dir('sample-app') {
                    sh 'mvn clean compile'
                }
            }
        }

        stage('Test') {
            steps {
                dir('sample-app') {
                    sh 'mvn test'
                }
            }
        }

        stage('Package') {
            steps {
                dir('sample-app') {
                    sh 'mvn package'
                }
            }
        }

        stage('Archive Artifact') {
            steps {
                archiveArtifacts artifacts: 'sample-app/target/*.war', fingerprint: true
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                echo 'Deploying WAR to Tomcat'

                dir('sample-app') {
                    sh '''
                    cp target/*.war /opt/tomcat/webapps/
                    systemctl restart tomcat10
                    '''
                }
            }
        }

        stage('Health Check') {
            steps {
                sh 'curl -f http://52.66.250.30:80/sample'
            }
        }
    }

    post {
        success {
            echo ' Pipeline completed successfully!'
        }
        failure {
            echo ' Pipeline failed!'
        }
    }
}
