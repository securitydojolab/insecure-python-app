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
                sh returnStatus: true, script: 'mkdir report'
            }
        }
    
      // Trufflehog Secrets Scanning
   stage ('Secret Scan') {
      steps {
        sh returnStatus: true, script: 'rm report/trufflehog.json'
        sh returnStatus: true, script: 'docker rm -f $(docker ps -a |  grep trufflehog |awk \'{print $1}\')'
        sh returnStatus: true, script: 'docker rmi $(docker images | grep trufflehog | awk \'{print $3}\') --force'
        sh 'docker run justmorpheu5/trufflehog https://github.com/justmorpheus/insecure-python-app --json > report/trufflehog.json'
        sh 'cat report/trufflehog.json'
      }
    }
            
   // Static Application Security Testing
    stage ('SAST Scan') {
      steps {
        sh returnStatus: true, script: 'rm report/bandit-result.json'
        sh returnStatus: true, script: 'docker rm -f $(docker ps -a |  grep bandit |awk \'{print $1}\')'
        sh returnStatus: true, script: 'docker rmi $(docker images | grep bandit | awk \'{print $3}\') --force'
        sh 'docker run --rm -v $(pwd):/bandit justmorpheu5/bandit -r . -f json  > report/bandit-result.json'
        sh 'cat report/bandit-result.json'
      }
    }

   // Source Composition Analysis Dependency Scanning Frontend
   stage ('SCA Frontend Scan') {
      steps {
        sh returnStatus: true, script: 'rm -f odc-reports/dependency-check-*'
        sh 'wget "https://raw.githubusercontent.com/justmorpheus/devsecops-tools/main/owasp-dependency-check.sh" '
        sh 'chmod +x owasp-dependency-check.sh'
        sh 'bash owasp-dependency-check.sh'
        sh 'cat odc-reports/dependency-check-report.csv'
       
      }
    }   
    // Source Composition Analysis Dependency Scanning Backened
   stage ('SCA Backend Scan') {
      steps {
        sh returnStatus: true, script: 'rm report/safety-result.json'
        sh returnStatus: true, script: 'docker rm -f $(docker ps -a |  grep safety |awk \'{print $1}\')'
        sh returnStatus: true, script: 'docker rmi $(docker images | grep safety | awk \'{print $3}\') --force'
        sh 'docker run --rm -v $(pwd):/src justmorpheu5/safety check -r requirements.txt --json  > report/safety-result.json'
        sh 'cat report/safety-result.json'
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
            
        stage('Beta Test Stage') {
           steps {
                sh label: '', script: "docker run -d --name ${JOB_NAME} -p 8000:8000 ${image}"
          }
        }
        
       stage('Update Dockerhub') {
            steps {
                script {
                    docker.withRegistry( 'https://registry.hub.docker.com ', registryCredential ) {
                        dockerImage.push()
                    }
                }
            }
      
        }
     
              stage('Deploy to Prod') {
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
