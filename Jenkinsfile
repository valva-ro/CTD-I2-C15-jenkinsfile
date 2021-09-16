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
            dir ('maven-adderapp') {
                script {
                    pom = readMavenPom file: "pom.xml";
                    files = findFiles(glob: "target/*.${pom.packaging}");
                    filePath = files[0].path;
                    nexusArtifactUploader (
                        nexusVersion: "3.33.1-01",
                        protocol: "http",
                        nexusUrl: "${env.NEXUS}:8081",
                        groupId: pom.groupId,
                        version: pom.version,
                        repository: "maven-jenkins",
                        credentialsId: "nexus",
                        artifacts: [
                            [
                                artifactId: pom.artifactId,
                                classifier: '',
                                file: filePath,
                                type: pom.packaging
                            ],
                            [
                                artifactId: pom.artifactId,
                                classifier: '',
                                file: "pom.xml",
                                type: "pom"
                            ]
                        ]
                    );
                }
            }
        }
   }
}
