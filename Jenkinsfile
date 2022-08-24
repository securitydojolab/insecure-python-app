def img
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
                    img = registry + ":${env.BUILD_ID}"
                    println ("${img}")
                    dockerImage = docker.build("${img}")
                }
            }
        }

    }
}
