repositoryName = "test"
env.BUILD_URL = "http://54.83.181.206:8081/artifactory/test/cjp-1.3-SNAPSHOT.jar"
env.JOB_NAME= "test"
pipeline {
	agent any
	stages {
	stage ('build') {
		steps {
			script {		
 	
             sh 'mvn clean package'
		}
	}
	}
if (branchName == "master") {
              promoteStage()
              }

         def promoteStage(){
              // Stage: promote
                   stage ('Appprove to proceed'){	
                       notifyQA()
	               proceedConfirmation("proceed1","promote to Prod ?")
                  }
                  node{
	           stage ('Promote artifacts to Prod'){
                   def server = Artifactory.server 'test'
                   def uploadSpec  =  """{
                   "files": [
              {
                                "pattern": "/var/lib/jenkins/workspace/test/target/cjp-1.3-SNAPSHOT.jar",
                               "target": "${repositoryName}" 
                            }
                         ]
                    }"""
                def buildInfo1 = server.upload(uploadSpec)
			server.publishBuildInfo(buildInfo1)
			}
		    }
		}

		def notifyQA(String buildStatus = 'STARTED') {
	        // build status of null means successful
	        buildStatus =  buildStatus ?: 'SUCCESSFUL'
	        def toList = qaEmailId
	        def subject = "QA: '${repositoryName}' artifact ready for promotion to Prod"
	        def summary = "${subject} (${env.BUILD_URL})"
	        def details = """
	        <p>Job '${env.JOB_NAME} [/var/lib/jenkins/workspace/test/target/cjp-1.3-SNAPSHOT.jar]' is ready to be promoted from DEV to QA.</p>
	        <p>Click here to move the library into the QA artifactory for testing. "<a href="${env.BUILD_URL}/input">${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>"</p>
	   """
	        emailext body: details,mimeType: 'text/html', subject: subject, to: toList
                }
		def proceedConfirmation(String id, String message) {	
                def userInput = true
                def didTimeout = false
                try {
                timeout(time: waitingTime, unit: 'HOURS') { //
                userInput = input(
                id: "${id}", message: "${message}", parameters: [
                [$class: 'BooleanParameterDefinition', defaultValue: true, description: '', name: 'Confirm to proceed !']
             ])
          }
      } 
		catch(e) { // timeout reached or input false
                def user = e.getCauses()[0].getUser()
                if('SYSTEM' == user.toString()) { // SYSTEM means timeout.
                didTimeout = true
                if (didTimeout) {
                echo "no input was received abefore timeout"
                currentBuild.result = "FAILURE"
                throw e
            } 
		else if (userInput == true) {
                echo "this was successful"
            } 
		else {
                userInput = false
                echo "this was not successful"
                currentBuild.result = "FAILURE"
                println("catch exeption. currentBuild.result: ${currentBuild.result}")
                throw e
            }
       } 
		else {
                userInput = false
                echo "Aborted by: [${user}]"
     }
   }
 } 
	}
}
	
   
