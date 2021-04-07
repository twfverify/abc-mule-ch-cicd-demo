pipeline {
  agent any
  stages {
  
    stage('Build Application') { 
      steps {
        echo 'Build the Mule Application...' 
        bat 'mvn clean install'
      }
    }
    
    stage('Packaging the Application'){
		steps {
			echo 'Packaging the Mule Application...' 
			bat 'mvn package -DskipTests'
		}
	}
    
 	stage('Perform MUnit Testing the Application') { 
      steps {
        echo 'Perform MUnit Testing the Mule Application...' 
        bat 'mvn test'
      }
    }
 	
   	stage('Deploy the Application into Mulesoft CloudHub Environment') { 
      environment {
        ANYPOINT_CREDENTIALS = credentials('anypoint.credentials')
      }
            
      steps {
        echo 'Deploying only because of code commit...'
        echo 'Deploying to  dev environment....'
        //bat 'mvn clean deploy -DmuleDeploy -Dusername=muser1 -Dpassword=Lakshmi1 -Dworkers=1 -Dworker.type=Micro -Denvironment=Sandbox -Dmule.version=4.3.0'
        bat 'mvn clean deploy -DmuleDeploy -DskipTests -Dusername=${ANYPOINT_CREDENTIALS_USR} -Dpassword=${ANYPOINT_CREDENTIALS_PSW} -DworkerType=Micro -Dworkers=1 -Denvironment=Sandbox -Dmule.version=4.3.0'
      }
	 }
	 
	 stage("Perform Regression Testing"){
			steps{
				echo 'Perform Regression Testing the Mule Application...' 
				//bat 'C:\\Users\\Administrator\\AppData\\Roaming\\npm\\newman run c:\\newman\\<postman collection>.json --diable-unicode'
			}
	 }
	 
	 stage("Moving the Artifact into Nexus Repository"){
		steps{
			echo 'Moving the Artifact into Nexus Repository...' 
			//bat 'mvn deploy -DskipTests=true -DskipITs=true'
		}
	}
	
  }
  
}