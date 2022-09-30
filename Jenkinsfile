def image
pipeline {
        environment {
        registry = "gurubaba/vuln-python" //To push an image to Docker Hub, you must first name your local image using your Docker Hub username and the repository name that you created through Docker Hub on the web.
        registryCredential = 'dockerlogin'
        dockerImage = ''
    }
    agent  any
    stages {
        stage ("Build Checkout") {
            steps {
                // Clean workspace before build
                cleanWs()
                git branch: 'main',
                    url: 'https://github.com/securitydojolab/insecure-python-app'

            }
        }
    stage ('Stop Old Container'){
            steps{
                sh returnStatus: true, script: 'docker stop $(docker ps -a | grep ${JOB_NAME} | awk \'{print $1}\')'
                sh returnStatus: true, script: 'docker rmi $(docker images | grep ${registry} | awk \'{print $3}\') --force' //this will delete all images
                sh returnStatus: true, script: 'docker rm ${JOB_NAME}'
                sh returnStatus: true, script: 'docker rm -f $(docker ps -a |  grep vuln-python |awk \'{print $1}\')'
                sh returnStatus: true, script: 'mkdir report'
            }
        }
    
      // Trufflehog Secrets Scanning
   stage ('Secret Scan') {
      steps {

        sh returnStatus: true, script: 'rm report/trufflehog.json'
        sh returnStatus: true, script: 'docker run justmorpheu5/trufflehog https://github.com/securitydojolab/insecure-python-app --json > report/trufflehog.json'
        sh returnStatus: true, script: 'docker rm -f $(docker ps -a |  grep trufflehog |awk \'{print $1}\')'
        sh returnStatus: true, script: 'docker rmi $(docker images | grep trufflehog | awk \'{print $3}\') --force'
        sh 'cat report/trufflehog.json'
      }
    }
   // Source Composition Analysis Dependency Scanning Frontend
   stage ('SCA Frontend Scan') {
      steps {
        sh returnStatus: true, script: 'rm -f odc-reports/dependency-check-*'
        sh returnStatus: true, script: 'rm /usr/share/dependency-check/data/odc.update.lock'
        sh 'wget "https://raw.githubusercontent.com/justmorpheus/devsecops-tools/main/owasp-dependency-check.sh" '
        sh 'chmod +x owasp-dependency-check.sh'
        sh 'mkdir odc-reports'
        sh 'bash owasp-dependency-check.sh'
        sh 'cat odc-reports/dependency-check-report.csv'
        sh returnStatus: true, script: 'docker rm -f $(docker ps -a |  grep dependency-check |awk \'{print $1}\')'
        sh returnStatus: true, script: 'docker rmi $(docker images | grep dependency-check | awk \'{print $3}\') --force'
       
      }
    }   
    // Source Composition Analysis Dependency Scanning Backened
   stage ('SCA Backend Scan') {
      steps {
        sh returnStatus: true, script: 'rm report/safety-report.json'
        sh returnStatus: true, script: 'docker run --rm -v $(pwd):/src justmorpheu5/safety check -r requirements.txt --json  > report/safety-report.json'
        sh returnStatus: true, script: 'docker rm -f $(docker ps -a |  grep safety |awk \'{print $1}\')'
        sh returnStatus: true, script: 'docker rmi $(docker images | grep safety | awk \'{print $3}\') --force'
        sh 'cat report/safety-report.json'
      }
    }
   // Static Application Security Testing
    stage ('SAST Scan') {
      steps {

        sh returnStatus: true, script: 'rm report/bandit-report.json'
        sh returnStatus: true, script: 'docker run --rm -v $(pwd):/bandit justmorpheu5/bandit -r . -f json  > report/bandit-report.json'
        sh returnStatus: true, script: 'docker rm -f $(docker ps -a |  grep bandit |awk \'{print $1}\')'
        sh returnStatus: true, script: 'docker rmi $(docker images | grep bandit | awk \'{print $3}\') --force'
        sh 'cat report/bandit-report.json'
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
                sh returnStatus: true, script: 'docker rm -f $(docker ps -a |  grep vuln-python |awk \'{print $1}\')'
                sh returnStatus: true, script: 'docker rmi $(docker images | grep buster | awk \'{print $1}\'):$(docker images | grep buster | awk \'{print $2}\') --force'
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
                    sshagent(['sshlogin']) {
                        // Change the IP address of the production server
                        sh returnStatus: true, script: "ssh -o StrictHostKeyChecking=no ubuntu@prod.securitydojo.co.in ${stop_container}"
                        sh returnStatus: true, script: "ssh -o StrictHostKeyChecking=no ubuntu@prod.securitydojo.co.in ${delete_contName}"
                        sh returnStatus: true, script: "ssh -o StrictHostKeyChecking=no ubuntu@prod.securitydojo.co.in ${delete_images}"

                    // Change the IP address of the production server
                        sh "ssh -o StrictHostKeyChecking=no ubuntu@prod.securitydojo.co.in ${image_run}"
                        archiveArtifacts artifacts: '**/*'
                    }
                }
            }
        }
     
       // Dynamic Application Security Testing
    stage ('DAST Scan') {
      steps {

        // Change the IP address of the production server
        sh returnStatus: true, script: 'rm report/nikto-report.xml'
        sh returnStatus: true, script: 'docker run --rm -u $(id -u):$(id -g) -v $(pwd):/tmp justmorpheu5/nikto -h http://devsecops.securitydojo.co.in -o /tmp/report/nikto-report.xml'
        sh returnStatus: true, script: 'docker rm -f $(docker ps -a |  grep nikto |awk \'{print $1}\')'
        sh returnStatus: true, script: 'docker rmi $(docker images | grep nikto | awk \'{print $3}\') --force'
        sh 'cat report/nikto-report.xml'
      }
    }
       // SSL Scan
    stage ('SSL Scan') {
      steps {

        // Change the google.com to production domain
        sh returnStatus: true, script: 'rm report/sslyze-report.json'
        sh returnStatus: true, script: 'docker run --rm -u $(id -u):$(id -g) -v $(pwd):/tmp justmorpheu5/sslyze devsecops.securitydojo.co.in --json_out /tmp/report/sslyze-report.json'
        sh returnStatus: true, script: 'docker rm -f $(docker ps -a |  grep sslyze |awk \'{print $1}\')'
        sh returnStatus: true, script: 'docker rmi $(docker images | grep sslyze | awk \'{print $3}\') --force'
        sh 'cat report/sslyze-report.json'
      }
    }
       // Nmap Network Scan
    stage ('Nmap Scan') {
      steps {

        // Change the IP address of the production server
        sh returnStatus: true, script: 'rm report/nmap-report.xml'
        sh returnStatus: true, script: 'docker run --rm -u $(id -u):$(id -g) -v $(pwd):/tmp justmorpheu5/nmap devsecops.securitydojo.co.in -oX /tmp/report/nmap-report.xml'
        sh returnStatus: true, script: 'docker rm -f $(docker ps -a |  grep nmap |awk \'{print $1}\')'
        sh returnStatus: true, script: 'docker rmi $(docker images | grep nmap | awk \'{print $3}\') --force'
        sh 'cat report/nmap-report.xml'
      }
    }
// OWASP ZAP Baseline Scan
    stage ('ZAP Baseline Scan') {
      steps {

        // Change the IP address of the production server
        sh returnStatus: true, script: 'docker run --user 0 --rm -v $(pwd):/zap/wrk:rw owasp/zap2docker-stable:2.11.1 zap-baseline.py -t http://devsecops.securitydojo.co.in -x zap-report.xml'
        sh returnStatus: true, script: 'docker rm -f $(docker ps -a |  grep zap |awk \'{print $1}\')'
        sh returnStatus: true, script: 'docker rmi $(docker images | grep zap | awk \'{print $3}\') --force'
      }
    }
 // Vulnerability Management
    stage ('Vulnerability Management') {
      steps {

        sh returnStatus: true, script: 'wget https://raw.githubusercontent.com/justmorpheus/devsecops-tools/main/upload-results.py'
              
        sh returnStatus: true, script: 'chmod +x upload-results.py'
        //Bandit Scan Report
        sh returnStatus: true, script: 'python3 upload-results.py --host 54.68.68.238:8080 --api_key 1e5d07659ca2a02cb2e43b8fd5fee6c859c0b328 --engagement_id 1 --product_id 1 --lead_id 1 --environment "Production" --result_file report/bandit-report.json --scanner "Bandit Scan"'
        //Trufflehog Scan Report
        sh returnStatus: true, script: 'python3 upload-results.py --host 54.68.68.238:8080 --api_key 1e5d07659ca2a02cb2e43b8fd5fee6c859c0b328 --engagement_id 1 --product_id 1 --lead_id 1 --environment "Production" --result_file report/trufflehog.json --scanner "Trufflehog Scan"'
        //Safety not supported By DefectDojo
        //SSLyze Scan Report
        sh returnStatus: true, script: 'python3 upload-results.py --host 54.68.68.238:8080 --api_key 1e5d07659ca2a02cb2e43b8fd5fee6c859c0b328 --engagement_id 1 --product_id 1 --lead_id 1 --environment "Production" --result_file report/sslyze-report.json --scanner "Sslyze Scan"'
        //Nikto Scan Report
        sh returnStatus: true, script: 'python3 upload-results.py --host 54.68.68.238:8080 --api_key 1e5d07659ca2a02cb2e43b8fd5fee6c859c0b328 --engagement_id 1 --product_id 1 --lead_id 1 --environment "Production" --result_file report/nikto-report.xml --scanner "Nikto Scan"'
        //Nmap Scan Report
        sh returnStatus: true, script: 'python3 upload-results.py --host 54.68.68.238:8080 --api_key 1e5d07659ca2a02cb2e43b8fd5fee6c859c0b328 --engagement_id 1 --product_id 1 --lead_id 1 --environment "Production" --result_file report/nmap-report.xml --scanner "Nmap Scan"'
        //Dependency Check Scan Report
        sh returnStatus: true, script: 'python3 upload-results.py --host 54.68.68.238:8080 --api_key 1e5d07659ca2a02cb2e43b8fd5fee6c859c0b328 --engagement_id 1 --product_id 1 --lead_id 1 --environment "Production" --result_file odc-reports/dependency-check-report.xml --scanner "Dependency Check Scan"'
        // OWASP Zap baseline Scan
        sh returnStatus: true, script: 'python3 upload-results.py --host 54.68.68.238:8080 --api_key 1e5d07659ca2a02cb2e43b8fd5fee6c859c0b328 --engagement_id 1 --product_id 1 --lead_id 1 --environment "Production" --result_file zap-report.xml --scanner "ZAP Scan"'
      // OWASP Zap baseline Scan
        sh returnStatus: true, script: 'python3 upload-results.py --host 54.68.68.238:8080 --api_key 1e5d07659ca2a02cb2e43b8fd5fee6c859c0b328 --engagement_id 1 --product_id 1 --lead_id 1 --environment "Production" --result_file zap-report.xml --scanner "ZAP Scan"'
       // Upload Reports To Gdrive
        // Upload Reports To Gdrive
        withCredentials([string(credentialsId: 'client_secrets.json', variable: 'CLIENT_SECRET')]) { //set SECRET with the credential content
        sh '''
        echo ${CLIENT_SECRET} > client_secrets.json
        '''
        }       
        withCredentials([string(credentialsId: 'mycred.txt', variable: 'SECRET')]) { //set SECRET with the credential content
        sh '''
        wget https://raw.githubusercontent.com/justmorpheus/devsecops-tools/main/gdrive_upload/client_secrets.json
        '''
        sh returnStatus: true, script: 'git clone https://github.com/justmorpheus/devsecops-tools.git'
        sh'''
        echo ${SECRET} > mycreds.txt
        '''
        sh returnStatus: true, script: 'python3 -m pip install -r devsecops-tools/gdrive_upload/requirements.txt && python3 devsecops-tools/gdrive_upload//main.py'
        }
              
      }
    }
    stage ('Infrastructure As A Code') {
      steps {

        sh returnStatus: true, script: 'git clone https://github.com/securitydojolab/devsecops-infrastructure'
        sh returnStatus: true, script: 'docker run --tty --volume /var/lib/jenkins/workspace/devsecops/devsecops-infrastructure:/tf --workdir /tf bridgecrew/checkov --directory /tf'
      }
    } 
    }

post {
       // only triggered when blue or green sign
       success {
           slackSend (color: "good", message: "DevSecops Build Success - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
       }
       // triggered when red sign
       failure {
           slackSend (color: "danger", message: "DevSecops Build Failed - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
       }
       
       aborted {
           slackSend (color: "#b72709", message: "Build Aborted - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
       }

    }
}
