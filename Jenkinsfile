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
     
   stage ('Check GitSecrets') {
      steps {
        sh returnStatus: true, script: 'rm trufflehog.json'
        sh returnStatus: true, script: 'docker rm -f $(docker ps -a |  grep trufflehog |awk \'{print $1}\')'
        sh returnStatus: true, script: 'docker rmi $(docker images | grep trufflehog | awk \'{print $3}\') --force'
        sh 'docker run trufflesecurity/trufflehog github --repo https://github.com/justmorpheus/insecure-python-app  > trufflehog.json'
        sh 'cat trufflehog.json'
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
                script {
                    def stop_container = "docker stop ${JOB_NAME}"
                    def delete_contName = "docker rm ${JOB_NAME}"
                    def delete_images = 'docker image prune -a --force'
                    def image_run = "docker run -d --name ${JOB_NAME} -p 8000:8000 ${image}"
                    println "${image_run}"
                    sshagent(['tomcat']) {
                        sh returnStatus: true, script: "ssh -o StrictHostKeyChecking=no ubuntu@35.89.66.74 ${stop_container}"
                        sh returnStatus: true, script: "ssh -o StrictHostKeyChecking=no ubuntu@35.89.66.74 ${delete_contName}"
                        sh returnStatus: true, script: "ssh -o StrictHostKeyChecking=no ubuntu@35.89.66.74 ${delete_images}"

                    // some block
                        sh "ssh -o StrictHostKeyChecking=no ubuntu@35.89.66.74 ${image_run}"
                        archiveArtifacts artifacts: '**/*'
                    }
                }
            }
        }
            

    }
}
