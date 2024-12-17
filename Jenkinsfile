pipeline {
    agent any
    tools {
        maven 'maven38'
    }

    environment {
        DOCKERHUB_CREDENTIALS=credentials('docker_token')
        DOCKERUSER="charityngenge"
    }   

    stages {
        
        stage('Credential Scanner for detecting Secrets') {
            steps {
                script {
                    def buildUrl = env.BUILD_URL
                    sh "gitleaks detect -v --no-git --source . --report-format json --report-path secrets.json || exit 0"
                }
            }
        }    
        stage('Build pet clinic') {
            steps {
                sh "mvn clean install"
            }
        }

        stage('Test Petclinic') {
            steps {
                script {
                    //Run Unit Test
                    sh 'mvn test'
                }
            }
            post {
                always {
                    //Archive and publish test results
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }
        stage('Package Petclinic') {
            steps {
                script {
                    // Package the application (e.g., create a JAR or WAR file)
                    sh 'mvn package'
                }
            }
            post {
                success {
                    // Archive the package artifact
                    archiveArtifacts artifacts: 'spring-petclinic-*/target/*.jar', allowEmptyArchive: true
                }
            }
        }
        stage('Docker Build Petclinic') {

			steps {
                script {
                    sh '''
                     def MICROSERVICE=${" spring-petclinic-admin-server  spring-petclinic-api-gateway spring-petclinic-config-server spring-petclinic-customers-service spring-petclinic-discovery-server spring-petclinic-vets-service spring-petclinic-visits-service"
                      for ARTIFACT_NAME in $MICROSERVICE; do
                          echo "$ARTIFACT_NAME"
                          docker build -t $DOCKERUSER/${ARTIFACT_NAME}:3.2.7 .
                   done
                   '''
                }
			}
		}
		stage('Login to Docker HUB') {

			steps {
				sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
			}
		}

		stage('Push Docker Image to Container Registry') {

			steps {
				sh 'docker push  $DOCKERUSER/spring-petclinic:${ARTIFACT_NAME}.jar '
            }
        }
    }
}       