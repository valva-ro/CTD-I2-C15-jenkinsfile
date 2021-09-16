# Paso a paso - Consigna 4

## En la VM
1. Instalar Docker Compose y Docker Engine
    ```bash
      # Esto es para Docker Engine
      sudo apt-get install \
        apt-transport-https \
        ca-certificates \
        curl \
        gnupg \
        lsb-release -y
      
      curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
      
      echo \
        "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
        $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
     
      sudo apt-get install docker-ce docker-ce-cli containerd.io

      # Esto para Docker Compose
      sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

      sudo chmod +x /usr/local/bin/docker-compose
    ```
2. Crear el archivo `docker-compose.yaml`
3. Dentro del archivo pegar: 
    ```yaml
    version: '3'
    services:
    nexus:
        container_name: 'nexus'
        image: 'docker.io/sonatype/nexus3:3.33.1'
        user: 'nexus'
        environment:
            BASE_URL: 'http://localhost:8081'
            NEXUS_SECURITY_RANDOMPASSWORD: 'false'
        ports:
            - '8081:8081'
        ulimits:
            nofile:
            soft: 65536
            hard: 65536
        volumes:
            - './nexus_data:/nexus-data'
    ```
4. Guardar y cerrar el archivo
5. Verificar que el archivo se haya guardado bien con `cat docker-compose.yaml`
6. Antes de hacer el `docker-compose up` tenemos que configurar la carpeta  `nexus_data` y cambiarle los permisos de acceso. Para eso ejecutamos el comando `chmod 777 nexus_data`.
7. Ahora sí levantamos el contenedor con `docker-compose up`
8. En algún momento la terminal debería tirarnos esto:
    ```bash
    -------------------------------------------------
    nexus |
    nexus | Started Sonatype Nexus OSS 3.33.1-01
    nexus |
    nexus | 
    -------------------------------------------------
    ```
    Y ahí ya podríamos abrir Nexus en el navegador


## En el navegador: Nexus
1. Para acceder a Nexus abrimos en el navegador [http://[ip de nuestra VM]:8081](http://localhost:8081)
2. Iniciamos sesión con:
    - Usuario: admin
    - Contraseña: admin123
3. Configuramos un usuario como dice el pdf de la [parte 4.1](https://github.com/valva-ro/CTD-I2-C15-jenkinsfile/blob/main/consignas/CI-Jenkinsfile-Parte-4.1.pdf) en la página 3


## En nuestro repositorio de git

1. Modificamos el Jenkinsfile que teníamos de la parte 3, reemplazamos lo que teníamos en `stage('Test')`
    ```bash
        stage('Test') {
            steps {
                dir ('maven-adderapp') {
                    sh 'mvn test'
                }
            }
        }
    ```
2. Reemplazamos lo que teníamos en `post { }` por:
    ```bash
        always {
            dir ('maven-adderapp') {
                junit 'target/surfire-reports/*.xml'
            }
        }
        success {
            dir ('maven-adderapp') {
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
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
    ```
    Debería quedarnos como [este](https://github.com/valva-ro/CTD-I2-C15-jenkinsfile/blob/main/Jenkinsfile).


## En el navegador: Jenkins

1. Para acceder a Jenkins abrimos en el navegador [http://[ip de nuestra VM]:8080](http://localhost:8080)
2. Si no carga verificar que esté corriendo con `sudo systemctl status jenkins`, si no está corriendo ejecutamos el comando `sudo systemctl start jenkins` y volvemos a verificar el estado
3. Con Jenkins abierto vamos a **Administrar Jenkins** → **Administrar Plugins** → Seleccionamos la pestaña **Todos los plugins**
4. Buscamos los plugins **Nexus Artifact Uploader** y **Pipeline Utility Steps**, los seleccionamos y elegimos la opción **Descargar ahora e instalar después de reiniciar** --ESTO ES MUCHO MUY IMPORTANTE--
5. Tildamos el checkbox que dice **Reiniciar Jenkins cuando termine la instalación y no queden trabajos en ejecución** --ESTO ES MUCHO MUY IMPORTANTE--
6. Ahora podemos seguir como dice el pdf de la [parte 4.2](https://github.com/valva-ro/CTD-I2-C15-jenkinsfile/blob/main/consignas/CI-Jenkinsfile-Parte-4.2.pdf) en la página 9