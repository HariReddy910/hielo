currentBuild.displayName="helio_app_v14.0-#"+currentBuild.number
pipeline{
    agent any
 //   options {
   //   timeout(time: 1, unit: 'MINUTES') 
    //     }
 
    stages{
        stage("Checkout-SCM")
        {
            steps{
                cleanWs()
                checkout([$class: 'GitSCM',
                branches: [[name: '*/master']], 
                doGenerateSubmoduleConfigurations: false,
                extensions: [[$class: 'CleanBeforeCheckout']], 
                submoduleCfg: [], 
                userRemoteConfigs: [[credentialsId: 'github_credentials', 
                url: 'https://github.com/HariReddy910/hielo.git']]])
                echo "Download finished form SCM"
            }
        }
        stage("Build")
        {
            steps{
                  withSonarQubeEnv('SonarQube') {
                 sh label: '', script: 'mvn package sonar:sonar'
                 echo "archeiving Artifacts" 
                 archiveArtifacts '**/*.war'
                  }     
            }
        }
        stage("Quality Gate") {
            steps {
              timeout(time: 1, unit: 'MINUTES') {
                waitForQualityGate abortPipeline: true
              }
            }
          }
        stage ('Java Code Coverage') {
				steps {
        			publishHTML([allowMissing: true, alwaysLinkToLastBuild: false, keepAll: true, reportDir: 'target/site/jacoco/', reportFiles: 'index.html', reportName: 'Code Coverage Report', reportTitles: 'Code Coverage Report'])
				}
			}
	
			stage ('Publishing Test Results') {
				steps {
					junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml'	
				}
			}
       
        stage("Deployment-AppServer"){
            steps{
                echo "hi"
                sh label: '', script: 'scp /var/lib/jenkins/workspace/helio/webapp/target/webapp.war ubuntu@172.31.26.148:/var/lib/tomcat9/webapps/helio_app_v14.0.war'
            }
        }
       
           stage('uploading artifacts to Jfrog artfactory') {
           steps {
              script { 
                 def server = Artifactory.server 'Artifactory-1'
                 def uploadSpec = """{ 
                   "files": [{
                       "pattern": "**/*.war",
                       "target": "Jfrog-harindra-artifactory"
    
                       
                    }]
                 }"""
                  server.upload(uploadSpec) 
     }     }    }  
    }
    
}
