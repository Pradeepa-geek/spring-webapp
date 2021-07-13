pipeline {
    agent any

	environment {
        EMAIL_TO = 'pradi.csp@gmail.com'
    }
    stages {
        stage('Build') {
            steps {
                // Get some code from a GitHub repository
                git 'https://github.com/Pradeepa-geek/spring-webapp.git'

                // Run Maven on a Unix agent.
                sh "mvn -Dmaven.test.failure.ignore=true clean package"

                // To run Maven on a Windows agent, use
                // bat "mvn -Dmaven.test.failure.ignore=true clean package"
            }
			
            post {
                // If Maven was able to run the tests, even if some of the test
                // failed, record the test results and archive the jar file.
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
    }
}
