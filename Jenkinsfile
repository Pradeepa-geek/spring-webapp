pipeline {
    agent any

	environment {
        EMAIL_TO = 'pradi.csp@gmail.com'
    }
    stages {
        stage('Checkout and Build') {
            steps {
                // Get some code from a GitHub repository
                git 'https://github.com/Pradeepa-geek/spring-webapp.git'
                sh "mvn -Dmaven.test.failure.ignore=true clean package"

            }
			
            post {
                success {
                    junit '**/target/surefire-reports/TEST-*.xml'
                }
				failure {
						emailext body: 'View Results at $BUILD_URL to find out errors. <br><br> ${CHANGES} <br><br> -------------------------------------------------- <br>${BUILD_LOG, maxLines=100, escapeHtml=false}', 
							to: "${EMAIL_TO}", 
							subject: 'Build failed in Jenkins: $PROJECT_NAME - #$BUILD_NUMBER'
				}
				unstable {
						emailext body: 'View Results at $BUILD_URL to find out errors. <br><br> ${CHANGES} <br><br> -------------------------------------------------- <br>${BUILD_LOG, maxLines=100, escapeHtml=false}', 
						to: "${EMAIL_TO}", 
						subject: 'Unstable build in Jenkins: $PROJECT_NAME - #$BUILD_NUMBER'
				}
				changed {
						emailext body: 'View Results at $BUILD_URL to find out errors.', 
						to: "${EMAIL_TO}", 
						subject: 'Jenkins build is back to normal: $PROJECT_NAME - #$BUILD_NUMBER'
				}
    
            }
        }
		 stage('SonaqrQube Analysis') {
            steps {
				withSonarQubeEnv('sonarscanner'){
				   sh "mvn sonar:sonar"
					}
				}
			post{
				always {
				 script{
				     timeout(time: 1, unit: 'HOURS') {
							def qg = waitForQualityGate()
							if (qg.status != 'OK') {
							emailext body: 'Sonar Analysis is here .View Results at $BUILD_URL to find out errors. <br><br> ${CHANGES} <br><br> -------------------------------------------------- <br>${BUILD_LOG, maxLines=100, escapeHtml=false}', 
							to: "${EMAIL_TO}", 
							subject: 'Sonar Analysis has failed in Jenkins: $PROJECT_NAME - #$BUILD_NUMBER'
							error "Pipeline aborted due to quality gate failure: ${qg.status}"
							}
						}
				 }
				}
				success {
					archiveArtifacts 'target/*.war'
                }
			}
		}
		
		stage('Upload Artfacts to Nexus') {
				steps{
					nexusArtifactUploader artifacts: [[artifactId: 'webapp', classifier: '', file: 'target/webapptest.war', type: 'war']], credentialsId: 'nexus', groupId: 'com.awstechguide', nexusUrl: 'localhost:8081', nexusVersion: 'nexus2', protocol: 'http', repository: 'http://localhost:8081/repositories/webapp/', version: '0.0.1-SNAPSHOT'
				}
		}
    }
}
