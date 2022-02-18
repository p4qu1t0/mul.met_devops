#!groovy
// version 0.9

def pipelineBranch = ""
def errorList = []

pipeline {
    
    agent {
        label "Jenkins Mulesoft Node"
    }
    
    environment {
        AWS_PROFILE="default"
        PATH="$PATH:/usr/local/bin"
    }
    
    tools {
        maven 'MVN-3.3.9'
    }
    
    options {
        timestamps()
        disableConcurrentBuilds()
    }

    parameters {

        //TODO: Treure projetName del package.json de l'atribut contextRoot
        choice (
            choices: 'TEST\nUSER\nPROD',
            description: 'Nginx environment',
            name: 'deploymentEnvironment'
        )
        
        string (
            name : 'distPath',
            defaultValue: '/tmp/',
            description: 'Project nameroot path'
        )

        booleanParam (
            name : 'debug',
            defaultValue: true,
            description: 'Show debug information'
        )
    }

    stages {

        stage ('Job debug information') {
            when {
                expression { params.debug }
            }
            steps {
                echo '------ENV------'
                sh 'printenv|sort'
                echo '---------------'
            }
        }

        stage ('Initilize pipelineBranch') {
            steps {
                script {
                    pipelineBranch = (env.BRANCH_NAME ? env.BRANCH_NAME : env.GIT_LOCAL_BRANCH)
                }
            }
        }

        stage ('Deploy to Cloudhub') {
            steps {
                script {
                    echo "Deploy to Cloudhub"
                    dir(params.distPath) {
                        /*
                        sh """
                            mvn deploy com.mulesoft.etc... ${deploymentEnvironment} 
                        """
                        */
                    }
                }
            }
        }
        
        stage ('Merge code to master') {
            steps {
                script {
                    echo "Merge code to master"
                    /*
                    TODO:
                    if("PROD".equals("${params.deploymentEnvironment}) {
                        dir(params.distPath) {
                            sh """
                                git checkout master
                                git merge develop
                                git push origin master
                            """
                        }
                    }
                    */
                }
            }
        }
    }

    post {
        failure {  
            echo "Errors: ${errorList}"
            emailext (
                to: 'correo@generali.com',
                from: 'JenkinsAWS@noexists.generali.com',
                mimeType: 'text/html', 
                body: "Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}console <br> ${errorList.join('<br>')}",  
                subject: "[${deploymentEnvironment}] ERROR CI: Project name -> ${env.JOB_NAME}",
                attachLog: true,
                compressLog: true
            )
        } 
    }
}

