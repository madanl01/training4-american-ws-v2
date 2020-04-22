pipeline {

//
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
      environment {
        ENVIRONMENT = 'Sandbox'
        APP_NAME = 'sandbox-training4-american-ws-ml'
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