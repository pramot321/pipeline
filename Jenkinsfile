branchName = "preprod"
qaEmailId ="cjptech12@gmail.com"
repositoryName = "test"
waitingTime = 24
pipeline{
	agent any
	stages {
		stage('Build'){
		    steps
                {
                    script
                    {
	sh 'mvn clean package'
                    }
                }
		}


	}
}

	

              if (branchName == "preprod") {
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
	               			echo 'Hi Cloudjournee'

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
	        <p>Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' is ready to be promoted from DEV to QA.</p>
	        <p>Click here to move the library into the QA artifactory for testing. "<a href="${env.BUILD_URL}/input">${env.JOB_NAME}[${env.BUILD_NUMBER}]/</a>"</p>
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
	
	
