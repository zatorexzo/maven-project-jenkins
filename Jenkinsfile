pipeline {
    agent any
    
    environment {
        repoApp= "https://github.com/zatorexzo/maven-project-jenkins.git"
        rama = "master"

        dateTime = sh(returnStdout: true, script: 'date +%Y%m%d-%H%M').trim()
        imagen = "zatorexzo/testing-jenkins:${dateTime}"
    }
    
    stages {
        // stage('PruebaENV') {
        //     steps {
        //         echo "${repoApp}"
        //         echo "${rama}"
        //         echo "Test ${dateTime}"
        //         echo "${imagen}"
        //     }
        // }
        stage('Build') {
            // post {
            //     success {
            //         archiveArtifacts 'codigo/webapp/target/*.war'
            //     }
            // }
            steps {
                dir(path: 'codigo') {
                git branch: "${rama}", credentialsId: 'github_credential', 
                            poll: false, url: "${repoApp}"
                sh 'mvn clean package -DskipTests'
                }
            }
        }

        stage('Test') {
            steps {
                sh 'echo \'Realización de algunos Test\''
            }
        }
        stage('Push build') {
            steps {
                sh 'echo \'Creo el contenedor\''
                dir(path: 'contenedor') {
                    withCredentials(bindings: [usernamePassword(credentialsId: 'github_credential', 
                                                passwordVariable: 'password', usernameVariable: 'usuario')]) {
                        git branch: "${rama}", credentialsId: 'github_credential', 
                                    poll: false, url: "${repoApp}"
                        sh '''
                            cp ../codigo/webapp/target/webapp.war .
                            git add .
                            git config --global user.email \'zatorexzo@gmail.com\'
                            git config --global user.name  \'zatorexzo\'
                            git commit -m \'Desde Jenkinsfile\'
                            git config --local credential.helper "!f() { echo username=\\$usuario; echo password=\\$password; }; f"
                            git push origin ${rama}
                        '''
                    }
                }
            }
        }
        stage('Docker Build') {
            steps {
                script {
                    git branch: "${rama}", credentialsId: 'github_credential', 
                                 poll: false, url: "${repoApp}"    
                    docker.build("${imagen}")
                }
            }
        } 
        stage('Pushing Docker Image to Dockerhub') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_credential') {
                        docker.image("${imagen}").push()
                        docker.image("${imagen}").push('latest')
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                sh 'docker stop hello-world | true'
                sh 'docker rm hello-world | true'
                sh "docker run --name hello-world -d -it -p 8080:8080 ${imagen}"
            }
        }

    }   // Fin stages 
        
} // fin pipeline
