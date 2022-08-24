def image
pipeline {
        environment {
        registry = "gurubaba/vuln-python" //To push an image to Docker Hub, you must first name your local image using your Docker Hub username and the repository name that you created through Docker Hub on the web.
        registryCredential = 'dockerhublogin'
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
    stage ('Stop Old Container'){
            steps{
                sh returnStatus: true, script: 'docker stop $(docker ps -a | grep ${JOB_NAME} | awk \'{print $1}\')'
                sh returnStatus: true, script: 'docker rmi $(docker images | grep ${registry} | awk \'{print $3}\') --force' //this will delete all images
                sh returnStatus: true, script: 'docker rm ${JOB_NAME}'
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
            
        stage('Beta Run Stage') {
           steps {
                sh label: '', script: "docker run -d --name ${JOB_NAME} -p 8000:8000 ${image}"
          }
        }
        
       stage('Push To DockerHub') {
            steps {
                script {
                    docker.withRegistry( 'https://registry.hub.docker.com ', registryCredential ) {
                        dockerImage.push()
                    }
                }
            }
      
        }
     stage('Deploy to Test Server') {
             
             steps {
           sshagent(['tomcat']) {
                    def stopcontainer = "docker stop ${JOB_NAME}"
                    def delcontName = "docker rm ${JOB_NAME}"
                    def delimages = 'docker image prune -a --force'
                    def img_run = "docker run -d --name ${JOB_NAME} -p 8000:8000 ${image}"
                    println "${img_run}"
                    sshagent(['docker-test']) {
                        sh returnStatus: true, script: "ssh -o StrictHostKeyChecking=no ubuntu@54.213.153.153 ${stopcontainer} "
                        sh returnStatus: true, script: "ssh -o StrictHostKeyChecking=no ubuntu@54.213.153.153 ${delcontName}"
                        sh returnStatus: true, script: "ssh -o StrictHostKeyChecking=no ubuntu@54.213.153.153 ${delimages}"

                    // some block
                        sh "ssh -o StrictHostKeyChecking=no ubuntu@54.213.153.153 ${img_run}"

              }      
           } 



            }
        }
    }
}
