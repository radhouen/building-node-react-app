#!/usr/bin/env groovy
def projectName = currentBuild.projectName
def version = env.BUILD_NUMBER
def buildTag = env.BUILD_TAG
def fileName = env.npmPack


pipeline {
   // define Agent 
   agent { 
    node { label 'master' }
     }
    // define environment variables
    environment {
        CI = 'true'
        JENKINS_CRUMB = 'curl user username:password "<jenkins-url>/crumbIssuer/api/xml?xpath=concat(//crumbRequestField, \":\",//crumb)"'
		
    }

    // List of stages 
    // checkout ---> install dependencies ---> build ----> SonarQube Code Quality -----> Test  ----> Publish Artifact
    stages {

        //=============Stage 1:  Checkout  Application  repository ===============
        stage("Checkout") {
            steps {
                load "environmentVariables.groovy"
                echo "${env.DEV_SCM_REPOSITORY}"
                echo "${env.DEV_SCM_BRANCH}"
                git(url: "${env.DEV_SCM_REPOSITORY}", branch: "${env.DEV_SCM_BRANCH}", poll: true)
            }
        }
        //============= Stage 2:  Install dependencies + Build Application  ===============
        stage("Install dependencies + Build Application ") {
	        steps {
	                echo "Installing dependencies ......."
	                //sh 'mvn org.codehaus.mojo:exec-maven-plugin:exec'
	               sh 'npm install'
		       echo "Building Applications ....."
		       sh 'npm install'
	            }
		}
        // ========== Stage3: SonarQube Code Quality ====================================================
	stage("SonarQube analysis") {
		steps {
			// to configure sonar scanner server follow this link:
		        // https://stackoverflow.com/questions/42763384/unknown-stage-section-withsonarqubeenv
			// withSonarQubeDev documentation: https://www.jenkins.io/doc/pipeline/steps/sonar/
			withSonarQubeEnv('SonarQubeDev') {
      			sh 'npm run sonar-scanner'
    			}
    		}
    	}

        // =============== Step 4: Define WaitForQualitygate should be true to go to next stage ==========
        stage("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                    // true = set pipeline to UNSTABLE, false = don't
                    // Requires SonarQube Scanner for Jenkins 2.7+
                    waitForQualityGate abortPipeline: true
                }
            }
        }
     //============= Stage 5: E2E react test =========================== 
         stage('Test') {
            steps {
                sh 'npm run test'
            }
        }
    //========== Stage 6: Publish Artifact ====================
        stage('Pack artefacts'){
            steps {
            script {
                def npmPack = sh(returnStdout:true, script:'npm pack').trim()
                env.npmPack = npmPack
            	sh "echo ${npmPack}"
            }   
            }
        }
        stage('Archive/Upload Artefact to Nexus'){
 
                steps{
                      nexusArtifactUploader(
						    nexusVersion: 'nexus3',
						    protocol: 'http',
						    nexusUrl: 'localhost:8081',
						    groupId: 'com.example',
						    version: version,
						    repository: 'DynamicsDeveloperReleases',
						    credentialsId: 'jenkins-nexus-authentication',
						    artifacts: [
						        [artifactId: projectName,
						         classifier: '',
						         file: fileName,
						         type: 'tgz']
						    			]
 									)
                                          }
                                       }
    								}

    post {
        always {
          cleanWs() 
          
 
        }
        
        success{
            
                sh 'git commit "package.json" -m'
            

        }

        
        failure {
             //mail to: 'someone@somewhere.com' , subject: "Status of pipeline: ${currentBuild.fullDisplayName}" , body: "${env.BUILD_URL} has result ${currentBuild.result}"
        	echo "${currentBuild.projectName} has failed at ${env.BUILD_URL}"
        	
        }
       
    }
        
    }
    
