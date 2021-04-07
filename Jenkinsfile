pipeline {
  agent any
  stages {
    stage('Build Application') { 
      steps {
        bat 'mvn clean install'
      }
    }
 	stage('Test') { 
      steps {
        echo 'Test Appplication...' 
        bat 'mvn test'
      }
    }
 	
   	stage('Deploy CloudHub') { 
      environment {
        ANYPOINT_CREDENTIALS = credentials('anypoint.credentials')
      }
            
      steps {
        echo 'Deploying only because of code commit...'
        echo 'Deploying to  dev environent....'
        //bat 'mvn clean package deploy -DmuleDeploy -Dusername=muser1 -Dpassword=Lakshmi1 -Dworkers=1 -Dworker.type=Micro -Denvironment=Sandbox -Dmule.version=4.3.0'
        bat 'mvn package deploy -DmuleDeploy -Dusername=${ANYPOINT_CREDENTIALS_USR} -Dpassword=${ANYPOINT_CREDENTIALS_PSW} -DworkerType=Micro -Dworkers=1 -Denvironment=Sandbox -Dmule.version=4.3.0’
      }
	  
	}
  }
}