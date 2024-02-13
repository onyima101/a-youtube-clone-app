pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        APP_NAME = "youtube-clone"
        RELEASE = "1.0.0"
        DOCKER_USER = "onyima101"
        DOCKER_PASS = 'dockerhub'
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
	    JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")        
    }
    stages {
        stage('clean workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/onyima101/a-youtube-clone-app.git/'
            }
        }
        stage("Sonarqube Analysis") {
            steps {
                withSonarQubeEnv('Sonarqube-Server') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=ndcc-project-1 \
                    -Dsonar.projectKey=ndcc-project-1'''
                }
            }
        }
        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonarqube-Token'
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('TRIVY FS SCAN') {
             steps {
                 sh "trivy fs . > trivyfs.txt"
             }
         }
        stage("Build & Push Docker Image") {
            steps {
                script {
                    docker.withRegistry('', DOCKER_PASS) {
                        docker_image = docker.build "${IMAGE_NAME}"
                    }
                    docker.withRegistry('', DOCKER_PASS) {
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
                }
            }
        }         
        //  stage("Docker Build & Push"){
        //      steps{
        //          script{
        //           withDockerRegistry(credentialsId: 'dockerhub', toolName: 'docker'){   
        //               sh "docker build -t youtube-clone ."
        //               sh "docker tag youtube-clone onyima101/youtube-clone:latest "
        //               sh "docker push onyima101/youtube-clone:latest "
        //             }
        //         }
        //     }
        // }
        // stage("Trivy Image Scan") {
        //     steps {
        //         script {
        //             sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image onyima101/youtube-clone:latest --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table > trivyimage.txt')
        //         }
        //     }
        // }        
        // stage("TRIVY Image Scan"){
        //     steps{
        //         sh "trivy image onyima101/youtube-clone:latest > trivyimage.txt" 
        //     }
        // }
	    // stage("Trigger CD Pipeline") {
        //     steps {
        //         script {
        //             sh "curl -v -k --user admin:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'ec2-35-153-184-88.compute-1.amazonaws.com:8080/job/ndcc-project-1-CD/buildWithParameters?token=gitops-token'"
        //         }
        //     }
        // }
    }        
        // stage('Deploy to Kubernets'){
        //     steps{
        //         script{
        //             dir('Kubernetes') {
        //               withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'kubernetes', namespace: 'dev', restrictKubeConfigAccess: false, serverUrl: '') {
        //               sh 'kubectl delete --all pods'
        //               sh 'kubectl apply -f deployment.yml'
        //               sh 'kubectl apply -f service.yml'
        //               }   
        //             }
        //         }
        //     }
        // }
    // post {
    //  always {
    //     emailext attachLog: true,
    //         subject: "'${currentBuild.result}'",
    //         body: "Project: ${env.JOB_NAME}<br/>" +
    //             "Build Number: ${env.BUILD_NUMBER}<br/>" +
    //             "URL: ${env.BUILD_URL}<br/>",
    //         to: 'nd.onyima@gmail.com',                              
    //         attachmentsPattern: 'trivyfs.txt,trivyimage.txt'
    //     }
    // }
}
