pipeline {
    agent { label 'slave-1' }
	
	environment {
        JAVA_HOME = '/usr/lib/jvm/java-17-openjdk-amd64'
        MAVEN_HOME = '/usr/share/maven'
        PATH = "${JAVA_HOME}/bin:${MAVEN_HOME}/bin:${env.PATH}"
    }
     
	stages {
	    stage('Checkout code') {
		    steps {
		        echo 'Checking out code'
                checkout scm
            }
        }
		
		stage('SetupJava17') {
		    steps {
			    echo 'Setting up Java17'
				sh 'sudo apt-get update'
				sh 'sudo apt-get install -y openjdk-17-jre-headless'
			}
		}
		
		stage('SetupMaven') {
		    steps {
			    echo 'setting up maven'
				sh 'sudo apt-get install -y maven'
			}
		}
		
		stage('BuildMaven') {
		    steps {
			    echo 'Building app with maven'
				sh 'mvn clean package'
			}
		}
		
		stage ('Upload Artifact') {
		    steps {
			    echo 'Uploading Artifact'
				archiveArtifacts artifacts: 'target/petclinic-0.0.1-SNAPSHOT.jar', allowEmptyArchieve: true
			}
		}
		
		stage ('Run application') {
		    steps {
			    echo 'Running application'
				sh 'nohup mvn spring-boot:run &'
				sleep(time: 15, unit: 'SECONDS')
				
				//Fetch IP address & display the accessible URL
				script {
				    def publicIP = sh(script: "curl -s https://checkip.amazonaws.com", returnStdout: true).trim()
					echo "The accessible ip address of the host is: http://${publicIP}:8080"
			}
		}
	}
		
		stage ('Validate application') {
		    steps {
			    echo 'Validating applicaton'
				script {
				    def response = sh(script: 'curl --write-out "%{http_code}" --silent --output /dev/null http://locathost:8080', returnStdout: true).trim()
					if (response == "200") {
					echo 'The app is running successfully'
					} else {
					echo "The app failed to start. HTTP response code: ${response}"
					currentBuild.result = 'FAILURE'
	                error ("The app did not start correctly")
                }
            }
        }
    }
        
        stage ('Wait for 15 seconds') {
            steps {
                echo 'Waiting for 15 seconds'
                sleep(time: 15, unit: 'SECONDS')
            }
        }

        stage ('Stopping the application')	{
            steps {
                echo 'Stopping the app'
                sh 'mvn spring-boot:stop'
            }
        }
	}

        post {
            always {
                echo 'Cleaning up'
                sh 'pkill -f "mvn spring-boot:run" || true'
            }
        }
	}
    		
		

      			
