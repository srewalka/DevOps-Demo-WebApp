pipeline {
    agent any
    
 	tools {
        maven 'Maven3.6.3'
   	 }
    
    stages {
        stage('Code Checkout') {
            steps {
                slackSend channel: 'alerts', message: 'Discovery phase pipeline test'
                slackSend channel: 'alerts', message: 'Project checkout from Git'
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/rahulrayala/DevOps-Demo-WebApp.git']]])
                slackSend channel: 'alerts', message: 'Git checkout complete'
            }
       }  
        stage('Sonarqube Scanner') {
            environment {
                scannerHome = tool 'sonarqube'
            }
            steps {
                slackSend channel: 'alerts', message: 'Static code analysis is in progress'
                withSonarQubeEnv('sonarqube') {
                    sh """${scannerHome}/bin/sonar-scanner"""
                }
               }
                slackSend channel: 'alerts', message: 'Static code analysis is complete'
                }
        } 
        
        stage('Build Project') {
            steps {
                slackSend channel: 'alerts', message: 'Building project...'
       
                //sh 'mvn compile'
                sh 'mvn -Dmaven.test.failure.ignore=true clean package'
		 slackSend channel: 'alerts', message: 'Build completed successfully'
            }
            post {
       always {
           jiraSendBuildInfo branch: 'DEV-4', site: 'devopssquad13.atlassian.net'
       }
            }
        }
        stage('Deploy To Test') {
            steps {
               slackSend channel: 'alerts', message: 'Deploy the Application to the Test environment'             
               deploy adapters: [tomcat8(url: 'http://104.197.198.124:8080/', credentialsId: 'tomcat', path: '' )], contextPath: '/QAWebapp', war: '**/*.war'
            
        }
            post {
       always {
           jiraSendDeploymentInfo environmentId: 'http://104.197.198.124:8080/', environmentName: 'http://104.197.198.124:8080/', environmentType: 'testing', issueKeys: ['DEV-4'], serviceIds: [''], site: 'devopssquad13.atlassian.net', state: 'successful'
               slackSend channel: 'alerts', message: 'Application deployed to Test environment'
       }
		    
            }
       }
       
       
       stage('Deploy Artifacts') {
                 steps{
			 slackSend channel: 'alerts', message: 'Deploying artifacts to Artifactory...'
			rtMavenRun ( 
				tool: 'Maven3.6.3',
				pom: 'pom.xml', 
				goals: 'clean install'
			)
				rtServer (
                    			id: 'ARTIFACTORY_SERVER',
                    			url: 'https://rahulrayala1.jfrog.io/artifactory',
                    			credentialsId: 'deploy'
                			)
                	rtMavenDeployer (
                    			id: 'MAVEN_DEPLOYER',
                    			serverId: 'artifactory',
                    			releaseRepo: 'libs-release-local',
                    			snapshotRepo: 'libs-snapshot-local'
                			)
                	rtMavenResolver (
                    			id: 'MAVEN_RESOLVER',
                    			serverId: 'artifactory',
                    			releaseRepo: 'libs-release-local',
                    			snapshotRepo: 'libs-snapshot-local'
                		)
			 rtPublishBuildInfo (
				 serverId: 'artifactory'
			 )	
			 slackSend channel: 'alerts', message: 'Deploying Artifacts Complete!'	 	
			}
		}
           
       stage('UI Test') {
            steps {
                slackSend channel: 'alerts', message: 'Starting UI Tests...'
                sh 'mvn -f functionaltest/pom.xml test'
                publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '\\functionaltest\\target\\surefire-reports', reportFiles: 'index.html', reportName: 'UI Test Report', reportTitles: ''])
            	slackSend channel: 'alerts', message: 'UI Tests Complete!'
            }
       }
    /* 
    stage('Performance Test'){
        steps{
        	slackSend channel: 'alerts', message: 'Starting Performance Test...'
           	blazeMeterTest credentialsId: 'Blazemeter', testId: '', workspaceId: ''
        	slackSend channel: 'alerts', message: 'Performance Test Complete!'   
        }
        }
*/
    stage ('Deploy To Prod') {
        steps {
        	slackSend channel: 'alerts', message: 'Starting deployment to Production...'
            sh 'mvn clean install -f pom.xml'
            deploy adapters: [tomcat8(credentialsId: 'tomcat', path: '', url: 'http://34.121.97.193:8080')], contextPath: 'ProdWebapp', war: '**/*.war'
        	slackSend channel: 'alerts', message: 'Production deployment complete!'
        }
         post {
       always {
           jiraSendDeploymentInfo environmentId: 'http://34.121.97.193:8080/', environmentName: 'http://34.121.97.193:8080/', environmentType: 'production', issueKeys: ['DEV-4'], serviceIds: [''], site: 'devopssquad13.atlassian.net', state: 'successful'
       }
            }
    }
    
    stage('Sanity Test') {
      steps {
      	slackSend channel: 'alerts', message: 'Starting Sanity Test...'
      sh 'mvn clean install -f Acceptancetest/pom.xml'
        publishHTML([
            allowMissing: false,
            alwaysLinkToLastBuild: false,
            keepAll: false,
            reportDir: '\\Acceptancetest\\target\\surefire-reports',
            reportFiles: 'index.html',
            reportName: 'Sanity Test Report'
          ])
          slackSend channel: 'alerts', message: 'Sanity Test Complete!'
        }
      } 
   }   
}
