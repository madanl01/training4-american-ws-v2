pipeline {

//
  agent any
  environment {    
    DEPLOY_CREDS = credentials('deploy-anypoint-user')
    MULE_VERSION = '4.1.4'
    BG = "Infosys"
    WORKER = "Micro"
  }
  stages {
    
	stage('Sonarqube') {
	    environment {
	        scannerHome = tool 'sonarscanner'
	    }
	    steps {
	        withSonarQubeEnv('sonarqube') {
	            bat "${scannerHome}/bin/sonar-scanner" 
	          }
	          
	        timeout(time: 10, unit: 'MINUTES') {
            waitForQualityGate abortPipeline: true
            }
	     }          
	        
	}
	
    
    stage('Build') {
      steps {
            bat 'mvn -B -U -e -V clean -DskipTests package'
      }
    }

    stage('Test') {
      steps {
          bat "mvn test"
      }
    }

     stage('Deploy Development') {
     	
     	when { branch 'develop'
     	}
     
      environment {
        ENVIRONMENT = 'Sandbox'
        APP_NAME = 'sandbox-training4-american-ws-ml01'
      }
      steps {
            bat 'mvn -U -V -e -B -DskipTests deploy -DmuleDeploy -Dmule.version="%MULE_VERSION%" -Danypoint.username="%DEPLOY_CREDS_USR%" -Danypoint.password="%DEPLOY_CREDS_PSW%" -Dcloudhub.app="%APP_NAME%" -Dcloudhub.environment="%ENVIRONMENT%" -Dcloudhub.bg="%BG%" -Dcloudhub.worker="%WORKER%"'
      }
    }
	  

	stage('Input') {
	    steps {
				timeout(time: 15, unit: "MINUTES") {
				input message: 'Do you want to approve the deploy in  production?', ok: 'Yes', submitter: 'admin'
				}
	    }
	}

       stage("Interactive_Input") {
            steps {
                script {
                    // Variables for input
                    def inputENV
                    def inputAPPNAME
                    // Get the input
                    def userInput = input(
                            id: 'userInput', message: 'Enter Deployment Environment & Application name:?',
                            parameters: [
                                    string(defaultValue: 'None',
                                            description: 'Deployment Environment',
                                            name: 'ENV'),
                                    string(defaultValue: 'None',
                                            description: 'Application name',
                                            name: 'APPNAME'),
                            ])

                    // Save to variables. Default to empty string if not found.
                    inputENV = userInput.ENV?:''
                    inputAPPNAME = userInput.APPNAME?:''

                    // Echo to console
                    echo("Deployment Environment: ${inputENV}")
                    echo("Application name: ${inputAPPNAME}")

                }
            }
        }

     stage('Deploy Production') {
     	
     	when { branch 'master'
	       tag pattern: "v\\d+", comparator: "REGEXP" }
     	}
      
      environment {
        ENVIRONMENT = 'Sandbox'
        APP_NAME = 'sandbox-training4-american-ws-ml01'
      }
      steps {	      
    
            bat 'mvn -U -V -e -B -DskipTests deploy -DmuleDeploy -Dmule.version="%MULE_VERSION%" -Danypoint.username="%DEPLOY_CREDS_USR%" -Danypoint.password="%DEPLOY_CREDS_PSW%" -Dcloudhub.app="%APP_NAME%" -Dcloudhub.environment="%ENVIRONMENT%" -Dcloudhub.bg="%BG%" -Dcloudhub.worker="%WORKER%"'
      }
    }
  }

  tools {
    maven 'M3'
  }
}
