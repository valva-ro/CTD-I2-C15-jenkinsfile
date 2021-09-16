pipeline {
    agent any
    tools {
        maven "maven-nodo-principal"
    }

    stages {
        stage('Build') {
            steps {
                dir ('maven-adderapp') {
                 sh 'mvn -DskipTests clean package'
                }
            }
        }
        stage('Test') {
            steps {
                dir ('maven-adderapp') {
                    sh 'mvn test'
                }
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying....'
            }
        }
    }
    post {
        always {
            dir ('maven-adderapp') {
                junit allowEmptyResults: true, testResults: 'target/surfire-reports/*.xml'
            }
        }
        success {
            dir ('maven-adderapp') {
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }
   }
}
