pipeline {
    agent any

    tools {
        maven 'maven'
    }

    stages {
        stage('SCM Checkout') {
            steps {
                checkout(scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/Gaurav1517/sonarqube-test.git']]))
            }
        }
        
        stage('Compile Source Code') {
            steps {
                dir('Devops/addressbook/addressbook_main') { // relative path to workspace
                    sh 'mvn compile'
                }
            }
        }
        
        stage('Test Source Code') {
            steps {
                dir('Devops/addressbook/addressbook_main') {
                    sh 'mvn test'
                }
            }
        }
        
        stage('Package Source Code') {
            steps {
                dir('Devops/addressbook/addressbook_main') {
                    sh 'mvn package'
                }
            }
        }
        stage('SonarQube Review'){
	       steps{
		        dir('Devops/addressbook/addressbook_main'){
		            withSonarQubeEnv(credentialsId: 'sonar-token', installationName: 'sonar') {
    				    sh "/var/lib/jenkins/workspace/sonar-demo/apache-maven-3.9.9/bin/mvn clean verify sonar:sonar -Dsonar.projectKey=test -Dsonar.projectName='test'"
			        }
		        }
	        }   
	    }

    }
}
