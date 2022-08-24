def image
pipeline {
        environment {
        registry = "justmorpheu5/vuln-python" //To push an image to Docker Hub, you must first name your local image using your Docker Hub username and the repository name that you created through Docker Hub on the web.
        registryCredential = 'docker-hub-login'
        dockerImage = ''
    }
    agent  any
    stages {
        stage ("Build Checkout") {
            steps {
                git branch: 'main',
                    url: 'https://github.com/justmorpheus/insecure-python-app.git'

            }
        }
        
        stage('Build Image') {
            steps {
                script {
                    image = registry + ":${env.BUILD_ID}"
                    println ("${image}")
                    dockerImage = docker.build("${image}")
                }
            }
        }
            
        stage('Test - Run Docker Container on Jenkins node') {
           steps {
                sh label: '', script: "docker run -d --name ${JOB_NAME} -p 8000:8000 ${image}"
          }
        }

    }
}
