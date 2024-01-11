pipeline {
    agent any
tools{
   jdk 'jdk17'
   nodejs 'node16'
  }
  environment {
    SCANNER_HOME = tool 'sonar-scanner'
        APP_NAME = "reddit-clone-pipeline"
        RELEASE = "1.0.0"
        DOCKER_USER = "mirali94"
        DOCKER_PASS = 'DockerHub-Token'
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
	  }

    stages {
        stage('CleanWorkspace') {
            steps {
                cleanWs()
            }
        }
        stage('Git CheckOut') {
            steps {
                git branch: 'main', url: 'https://github.com/Mir9438/a-reddit-clone.git'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube-Server') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Reddit-Clone-CI \
                    -Dsonar.projectKey=Reddit-Clone-CI'''
                }
            }
        }
        stage('Quality Gate') {
            steps {
                waitForQualityGate abortPipeline: false, credentialsId: 'SonarQube-Token'
            }
        }
         stage('Install Dependancies') {
            steps {
                 sh "npm install"
            }
        }
         stage('Trivy FS Scan') {
            steps {
                 sh "trivy fs . > trivyfs.txt"
            }
        } 
	stage("Build & Push Docker Image") {
             steps {
                 script {
                     docker.withRegistry('',DOCKER_PASS) {
                         docker_image = docker.build "${IMAGE_NAME}"
                     }
                     docker.withRegistry('',DOCKER_PASS) {
                         docker_image.push("${IMAGE_TAG}")
                         docker_image.push('latest')
                     }
                 }
             }
         }
       stage("Trivy Image Scan") {
             steps {
                 script {
	              sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image mirali94/reddit-clone-pipeline --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table > trivyimage.txt')
                 }
             }
         }
	 stage ('Cleanup Artifacts') {
             steps {
                 script {
                      sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                      sh "docker rmi ${IMAGE_NAME}:latest"
                 }
             }
         }
	 post {
           always {
             emailext attachmentsPattern: 'trivyfs.txt,trivyimage.txt',
                  attachLog: true,
                  subject: "'${currentBuild.result}'",
                  body: "Project: ${env.JOB_NAME}<br/>" +
                        "Build Number: ${env.BUILD_NUMBER}<br/>" +
                        "URL: ${env.BUILD_URL}<br/>",
                  to: 'mir.ali19912@gmail.com'
               }
         }

    }
}
