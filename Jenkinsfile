@Library('devops-projects-shared-lib') _

pipeline {
    
    agent {
        label 'docker'
    }

    tools {
        nodejs 'NodeJS-Latest'
    }


    parameters {
        choice choices: ['create', 'delete'], description: 'Choose create or Delete', name: 'action'
        string defaultValue: 'nextflix', description: 'Name of the Image', name: 'ImageName'
        string defaultValue: 'v1', description: 'Tag of the Image', name: 'ImageTag'
        string defaultValue: 'csnkarthik', description: 'Name of the App', name: 'dockerHubUser'
    }
    environment {
        //DOCKER_CREDENTIALS = credentials('docker_credentials')

        SCANNER_HOME = tool 'sonar-scanner'

    }

    stages {
        stage('Git Checkout'){            
            when { expression { params.action == 'create' } }
            steps {                
                gitCheckout(
                    branch: 'main',
                    url: 'https://github.com/csnkarthik/DevSecOps-Project.git'
                )
            }
        }       
        
        stage('sonarQuebe Analysis'){
            when { expression { params.action == 'create' } }            
            steps {       
                script
                {
                    withSonarQubeEnv(credentialsId: 'sonar-api') 
                    {
                         sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
                    -Dsonar.projectKey=Netflix -X '''
                    }
                }  
                
            }
        }
        // stage("quality gate") {
        //     steps {
        //         script {
        //             waitForQualityGate abortPipeline: false, credentialsId: 'sonar-api'
        //         }
        //     }
        // }

         stage('quality gate'){
            when { expression { params.action == 'create' } }
            steps {         
                script{
                    //def sonarcubeCredentials = 'sonar-api'
                    qualityGateStatus('sonar-api');
                }      
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }

        stage('trivy fs scan') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }

        stage('Docker image build & push'){
            when { expression { params.action == 'create' } }            
            steps {         
                script
                {
                    dockerImagePush("${params.ImageName}", "${params.ImageTag}", "${params.dockerHubUser}");
                }
            }
        }

        stage('trivy image Scan'){
            when { expression { params.action == 'create' } }            
            steps {         
                script{
                    dockerImageScan("${params.ImageName}", "${params.dockerHubUser}");
                }
            }
        }

        stage('Deploy to container'){
            steps{
                sh 'docker run -d --name netflix -p 8081:80 nasi101/netflix:latest'
            }
        }        
        
    }
    
}