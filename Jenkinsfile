import groovy.json.JsonSlurperClassic

def jsonParse(def json) {
    new groovy.json.JsonSlurperClassic().parseText(json)
}
pipeline {
    agent any
    environment {
        channel='D044YFG2N5U' //el id de mi canal personal de slack
        NEXUS_PASSWORD     = credentials('nexus-password') //busca en la seccion credentials de jenkins user credential el que coincide con el nombre y guarda la pwd en la variable global
    }
    stages {
             
        stage("Paso 1: Build && Test"){
            steps {
                script{
                    sh "echo 'Build && Test!'"
                    sh "./mvnw clean package -e"    
                }
            }
        }

        stage("Paso 2: Sonar - Análisis Estático"){
            steps {
                script{
                    sh "echo 'Análisis Estático!'"
                        withSonarQubeEnv('sonarqube') {
                            sh "echo 'Calling sonar by ID!'"
                            // Run Maven on a Unix agent to execute Sonar.
                            sh './mvnw clean verify sonar:sonar -Dsonar.projectKey=ejemplo-maven-full-stages -Dsonar.projectName=ejemplo-maven-full-stages -Dsonar.java.binaries=build'
                        }
                        
                }
            }
        }
        
        stage("Paso 3: Curl Springboot maven sleep 20"){
            steps {
                script{
                    sh "nohup bash ./mvnw spring-boot:run  & >/dev/null"
                    sh "sleep 10 && curl -X GET 'http://localhost:8081/rest/mscovid/test?msg=testing'"
                    sh "newman run ejemplo-maven.postman_collection.json"
                }
            }
        }
        stage("Paso 4: Detener Spring Boot"){
            steps {
                script{ //pidof lista los procesos java ejecutandose y awk indica cual es el nro del proceso el kill -9 mata el proceso java
                    sh ''' 
                        echo 'Process Spring Boot Java: ' $(pidof java | awk '{print $1}')  
                        sleep 20
                        kill -9 $(pidof java | awk '{print $1}')
                    '''
                }
            }
        }
           stage("Paso 5: Subir Artefacto a Nexus!"){
            steps {
                script{
                    nexusPublisher nexusInstanceId: 'nexus',
                        nexusRepositoryId: 'maven-ceres-repository',
                        packages: [
                            [$class: 'MavenPackage',
                                mavenAssetList: [
                                    [classifier: '',
                                    extension: 'jar',
                                    filePath: 'build/DevOpsUsach2020-0.0.1.jar'
                                ]
                            ],
                                mavenCoordinate: [
                                    artifactId: 'DevOpsUsach2020',
                                    groupId: 'com.devopsusach2020',
                                    packaging: 'jar',
                                    version: '0.0.1'
                                ]
                            ]
                        ]
                }
            }
        }
        stage("Paso 6: Descargar Nexus"){
            steps {
                script{ //debería haber un token para el curl, no con password, esto funciona porque habilitamos el acceso anonimo a nexus
                    sh ' curl -X GET -u admin:admin "http://nexus:8081/repository/maven-ceres-repository/com/devopsusach2020/DevOpsUsach2020/0.0.1/DevOpsUsach2020-0.0.1.jar" -O'
                }
            }
        }
         stage("Paso 7: Levantar Artefacto Jar en server Jenkins"){
            steps {
                script{
                    sh 'nohup java -jar DevOpsUsach2020-0.0.1.jar & >/dev/null'
                }
            }
        }
          stage("Paso 8: Testear Artefacto - Dormir(Esperar 20sg) "){
            steps {
                script{
                    sh "sleep 20 && curl -X GET 'http://localhost:8081/rest/mscovid/test?msg=testing'"
                }
            }
        }
        stage("Paso 9:Detener Atefacto jar en Jenkins server"){
            steps {
                sh '''
                    echo 'Process Java .jar: ' $(pidof java | awk '{print $1}')  
                    sleep 20
                    kill -9 $(pidof java | awk '{print $1}')
                '''
                sh "echo 'Proceso killeado :)'"
            }
        }
    }
}
