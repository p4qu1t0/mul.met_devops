#!groovy
// version 0.9

def repos = []
def pipelineBranch = ""
def workspacePath = ""
def jenkinsConfigJson = ""

def getProjectRepo(workspacePath, projectURL) {
    
    echo "Checkout repo ${projectURL}."
    script {
        dir("${workspacePath}") {
            checkout changelog: false, poll: false, scm: [$class: 'GitSCM',
                                branches: [[name: '*/${feature}']], 
                                doGenerateSubmoduleConfigurations: false, 
                                extensions: [[$class: 'CloneOption', honorRefspec: false, noTags: true, shallow: false]], 
                                submoduleCfg: [], 
                                userRemoteConfigs: [[credentialsId: "varqjen-at-NMDAWS-Codecommit", url: "${projectURL}"]]]
        }
     }
}


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

        string (
            name : 'feature',
            defaultValue: 'develop',
            description: 'Feature code to deliver on virtual environment'
        )
        
        choice (
            choices: 'TEST\nUSER\nPROD',
            description: 'Environment to deploy',
            name: 'deploymentEnvironment'
		)
        
        
        string (
            name : 'repoURL',
            defaultValue: 'https://git-codecommit.eu-west-1.amazonaws.com/v1/repos/mul.test_api',
            description: 'Project git URL'
        )
        
        /*
        choice (
            choices: '\nmajor\nminor\npatch',
            description: 'semVersion style tag version increment',
            name: 'tagVersion'
		)
		*/

		booleanParam (
            name : 'runUnitTests',
            defaultValue: false,
            description: 'Run Unit Tests'
        )
        
		booleanParam (
            name : 'runE2ETests',
            defaultValue: false,
            description: 'Run e2e Tests'
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
                
                sh """
                    git --version
                	mvn --version
                	java --version 
                """
            }
        }

        stage ('Initilize pipelineBranch') {
            steps {
                script {
                    pipelineBranch = (env.BRANCH_NAME ? env.BRANCH_NAME : env.GIT_LOCAL_BRANCH)
                    workspacePath = "$WORKSPACE/${env.BUILD_ID}" 
                    echo "workspacePath: $workspacePath"
                }
            }
        }
        
        stage ('Download feature java source from SCM') {
            steps {
                script {
                    getProjectRepo(workspacePath, params.repoURL)
                    //jenkinsConfigJson = readJSON file: "$workspacePath/jenkins.config.json"
                }
            }
        }

        stage ('Build project') {
            steps {
                echo "Build project"
            	dir(workspacePath) {
            	   /*
	            	sh """
	            		mvn ...
	            	"""
	            	*/
            	}
            }
        }

		stage('Unit tests') {
            when {
                expression { params.runUnitTests }
            }
		    steps {
                echo "Build project"
                /* dir(workspacePath) {
		          sh "mvn test...."
		        }*/
		    }
		}

		stage('Code Analysis') {
            when {
                expression { params.runE2ETests }
            }
		    steps {
		        dir(workspacePath) {
		          echo "Code Analysis"
		          /*
		           sh "sonar:sonar -Dsonar.login=myAuthenticationToken"
		          */
		        }
		    }
		}
		/*
		stage ('Tag version') {
			when {
                expression { params.tagVersion }
			}
			steps {
				dir(workspacePath) {
	            	sh """
		                npm --no-git-tag-version version ${params.tagVersion}
	            	"""
            	}	
			}
        }
		*/
        
        stage ('Publish app') {
        	// -- Configura variables
        	steps {

                build (job: "mul-deploy-mulesoft-project/${pipelineBranch}", // Nombre del job de Jenkins de depliegue
                    parameters: [
                                 [$class: 'StringParameterValue',  name: 'deploymentEnvironment', value: "${deploymentEnvironment}"],
                                 [$class: 'StringParameterValue',  name: 'distPath',              value: "${workspacePath}"],
                                 [$class: 'BooleanParameterValue', name: 'debug',                 value: "${debug}"]
                                ])
        	}
        }
    }
}

