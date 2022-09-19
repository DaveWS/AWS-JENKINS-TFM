pipeline {
    agent any
       triggers {
        pollSCM "* * * * *"
       }
    stages {
        stage('Build Application') { 
            steps {
                echo '=== Consrucion de Aplicacion Petclinic ==='
                sh 'mvn -B -DskipTests clean package' 
            }
        }
        stage('Test Application') {
            steps {
                echo '=== Pruebas de Aplicacion Petclinic ==='
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        stage('Build Docker Image') {
            when {
                branch 'main'
            }
            steps {
                echo '=== Contrucion de la Imagen Docker ==='
                script {
                    app = docker.build("daveid01/petclinic-spinnaker-jenkins")
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'main'
            }
            steps {
                echo '=== Publicacion de la Imagen Docker ==='
                script {
                    GIT_COMMIT_HASH = sh (script: "git log -n 1 --pretty=format:'%H'", returnStdout: true)
                    SHORT_COMMIT = "${GIT_COMMIT_HASH[0..7]}"
                    docker.withRegistry('https://registry.hub.docker.com/repository/docker/daveid01/test2tfm', 'dockerHubCredentials') {
                        app.push("$SHORT_COMMIT")
                        app.push("latest")
                    }
                }
            }
        }
        stage('Remove local images') {
            steps {
                echo '=== Borrado de las imagenes locales ==='
                sh("docker rmi -f daveid01/petclinic-spinnaker-jenkins:latest || :")
                sh("docker rmi -f daveid01/petclinic-spinnaker-jenkins:$SHORT_COMMIT || :")
            }
        }
    }
}
