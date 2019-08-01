```
#!/usr/bin/env groovy
pipeline {
    agent any
    stages {
        stage('Build'){
            steps {
                echo 'Building...'
                script {
                    /**
                     * login to docker for private repository
                     * create credentials in jenkins page.
                     **/
                     withCredentials([usernamePassword(credentialsId: 'docker-login-creds', passwordVariable: 'password', usernameVariable: 'username')]){
                         sh '''
                            echo "${password} | docker login -u ${username} --password-stdin"
                         '''
                         def app = docker.build("docker-image")
                         app.push("latest")
                     }
                }
            }
        }
        stage('Test'){
            steps {
                echo 'Testing...' 
            }
        }
        stage('Deploy'){
            steps {
                sh "exit"
                withCredentials([usernamePassword(credentialsId: 'docker-login-creds', passwordVariable: 'password', usernameVariable: 'username')]){
                    /**
                    * Restart docker server
                    **/
                    sh '''
                        echo "${password} | docker login -u ${username} --password-stdin"
                        docker stop docker_image
                        docker rm docker_image
                        docker pull docker_image:latest
                        docker run -d -p 80:80 --name docker-image-name -t docker_image:latest
                    '''
                }
            }
        }
    }
}
```