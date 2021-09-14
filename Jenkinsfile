pipeline {
    agent any
    tools {
        maven "maven-nodo-principal"
    }

    stages {
        stage('Build') {
            steps {
                dir (‘maven-adderapp’) {
                 sh 'mvn -DskipTests clean package'
                }
            }
        }
        stage('Test') {
            steps {
                echo 'Testing..'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying....'
            }
        }
    }
    post {
        success {
            dir (‘maven-adderapp’) {
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }
   }
}
