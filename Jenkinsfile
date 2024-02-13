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
        DOCKER_CREDENTIALS = credentials('docker_credentials')
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

        // stage('Docker Image build'){
        //     when { expression { params.action == 'create' } }            
        //     steps {         
        //         script{
        //             dockerBuild("${params.ImageName}", "${params.ImageTag}", "${params.dockerHubUser}");
        //         }
        //     }
        // }

        stage('Docker Image Scane'){
            when { expression { params.action == 'create' } }            
            steps {         
                script{
                    dockerImageScan("${params.ImageName}", "${params.dockerHubUser}");
                }
            }
        }

        stage('Docker Image push'){
            when { expression { params.action == 'create' } }            
            steps {         
                script{
                    dockerImagePush("${params.ImageName}", "${params.ImageTag}", "${params.dockerHubUser}");
                }
            }
        }
       
    }
}