pipeline {
    agent any
	tools {
	maven "mvn"
	jdk "jdk8"
	}
	environment {
		registry = "akshaysr1993/dockerimage01"
		ARTVERSION = "${env.BUILD_ID}"
		registryCredential = 'dockerhub'
	}
    stages{
        stage('Fetch Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Akshay27031993/cicd-kube-docker.git'
            }
        }

        stage('BUILD'){
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }

        stage('UNIT TEST'){
            steps {
                sh 'mvn test'
            }
        }
		stage('Checkstyle Analysis'){
		    steps {
			    sh 'mvn checkstyle:checkstyle'
			}
		}
		stage('sonar Analysis'){
			environment {
				scannerHome = tool 'sonar4.7'
			}
			steps{
				withSonarQubeEnv('sonar'){
				sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
					-Dsonar.projectName=vprofile-repo \
					-Dsonar.projectVersion=1.0 \
					-Dsonar.sources=src/ \
					-Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
					-Dsonar.junit.reportsPath=target/surefire-reports/ \
					-Dsonar.jacoco.reportsPath=target/jacoco.exec \
					-Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
				}
			}
		}
		stage('Build Docker App Image') {
			steps {
			  script {
			  dockerImage = docker.build registry + ":$BUILD_NUMBER"
			  }
			}
		}
		stage('Upload Image') {
		  steps {
		    docker.withRegistry('', registryCredential) {
			dockerImage.push("V$BUILD_NUMBER")
			dockerImage.push("latest")
			}
		  }
		}
		stage('Remove Docker Image') {
			steps {
				sh "docker rmi $registry:V$BUILD_NUMBER"
			}
		}
		stage('Kubernetes Deploy'){
		  agent {label 'KOPS'}
		  steps{
		    sh "sudo upgrade --install --force vprofile-stack helm/vprofilecharts --set appimage=${registry}:V${BUILD_NUMBER} --namespace prod"
		  }
		}
	}
}
