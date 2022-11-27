pipeline {
    agent any
    // {
    //   node {
     //      label 'slavenode123'
     //  }
//    }
    environment {
        AWS_DEFAULT_REGION = "us-east-2"
	NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "44.192.101.29:8081"
        NEXUS_REPOSITORY = "Jaganrepository"
        NEXUS_CREDENTIAL_ID = "nexusid"
    }
	
    options {
        buildDiscarder logRotator(artifactDaysToKeepStr: '', 
        artifactNumToKeepStr: '', 
        daysToKeepStr: '2', 
        numToKeepStr: '2')
    }
    triggers {
        githubPush();
    }
    stages {
        stage('SCM') {
            steps {
                git branch: 'main', 
                credentialsId: '9a2ad51c-19dd-4509-98ce-5e58fd2848c6', 
                url: 'https://github.com/jagan3780/SampleApplication'
                slackSend channel: '#test', color: '#439FE0', message: 'Code downloaded to workspace folder', teamDomain: 'CloudzDevops', tokenCredentialId: 'slackid'
                
            }
        }
      stage("build & SonarQube analysis") {
            steps {
                sh 'mvn clean deploy'
                slackSend channel: '#test', color: 'danger', message: 'hello'
            } 

          } 
 /* stage("build & SonarQube analysis") {
            steps {
              withSonarQubeEnv('SonarQubeServer'){
                sh 'mvn clean sonar:sonar'
              }
            }
        }
        stage("Quality Gate") {
           steps{
            script {
              timeout(10) {
                def qg = waitForQualityGate()
                if (qg.status != 'OK') {
                    error "Pipeline aborted due to quality gate failure: ${qg.status}"
                }
                else {
                    sh 'mvn clean deploy'
                    echo "sucess-jagan"
                }
              }
            }
          }
        } 

	    
	    stage("upload in nexus") {
		    steps {
			    script { 
			     nexusArtifactUploader artifacts: [
				     [
					     artifactId: 'maven-web-application', 
					     classifier: '', 
					     file: 'target/maven-web-application', 
					     type: 'war'
				     ]
			     ], 
				     credentialsId: 'nexusid', 
				     groupId: 'com.mt', 
				     nexusUrl: '44.192.101.29:8081', 
				     nexusVersion: 'nexus3', 
				     protocol: 'http', 
				     repository: 'Jaganrepository', 
				     version: 'com.mt'
			    }
		    }
	    }  */
	    
	    
	    
	    
	    
	    stage("Publish to Nexus Repository Manager") {
            steps {
                script {
                    def pom = readMavenPom file: "pom.xml";
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    artifactPath = filesByGlob[0].path;
                    artifactExists = fileExists artifactPath;
                    if(artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}";
                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: pom.version,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                [
					artifactId: pom.artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: pom.packaging
				],
                                [
					artifactId: pom.artifactId,
                                classifier: '',
                                file: "pom.xml",
                                type: "pom"
				]
                            ]
                        );
                    } else {
                        error "*** File: ${artifactPath}, could not be found";
                    }
                }
            
        
	    
	    
	   
	    
	    
/*        stage('upload in S3'){
            steps{
                script {
                //timeout(10) {
                    if (fileExists('target/maven-web-application.war')) {
                        s3Upload consoleLogLevel: 'INFO', 
						dontSetBuildResultOnFailure: false, 
						dontWaitForConcurrentBuildCompletion: false, 
						entries: [[bucket: 'binaryfiles1', 
						excludedFile: '', 
						flatten: false, 
						gzipFiles: false, 
						keepForever: false, 
						managedArtifacts: false, 
						noUploadOnFailure: true, 
						selectedRegion: 'us-east-2',
						showDirectlyInBrowser: false, 
						sourceFile: 'target/maven-web-application.war', 
						storageClass: 'STANDARD', 
						uploadFromSlave: true, 
						useServerSideEncryption: false, 
						userMetadata: [[key: 'Type', value: 'Java']]]], 
						pluginFailureResultConstraint: 'FAILURE', 
						profileName: 'binaryfiles1', 
						userMetadata: []
                   }
                  
                   else {
                      echo "fail-jagan"
                        abortPipeline: true
                    }
                }
         }
     }   
        
      stage('Deploy / Update Environment'){
            steps{
            	withCredentials([string(credentialsId: 'ACCESS_KEY', variable: 'ACCESS_KEY')]) {
            		sh '''/usr/local/bin/aws configure set aws_access_key_id ${ACCESS_KEY}'''
            	}
            	withCredentials([string(credentialsId: 'SECRET_KEY', variable: 'SECRET_KEY')]) {
            		sh '''/usr/local/bin/aws configure set aws_secret_access_key ${SECRET_KEY}'''
            	}
                sh '''/usr/local/bin/aws configure set region $AWS_DEFAULT_REGION'''
                sh '''   
                    /usr/local/bin/aws elasticbeanstalk check-dns-availability --cname-prefix my-cname
                    /usr/local/bin/aws elasticbeanstalk create-application --application-name my-sample-application --description "my sample application"
                    /usr/local/bin/aws elasticbeanstalk create-application-version --application-name my-sample-application --version-label v1234 --source-bundle S3Bucket="binaryfiles1",S3Key="maven-web-application.war" --process

                    /usr/local/bin/aws elasticbeanstalk create-environment --application-name my-sample-application --environment-name my-sample-env --cname-prefix my-cname --version-label v1234 --solution-stack-name "64bit Amazon Linux 2 v4.3.1 running Tomcat 8.5 Corretto 8" --option-settings Namespace="aws:autoscaling:launchconfiguration",OptionName="IamInstanceProfile",Value="aws-elasticbeanstalk-ec2-role"
                '''
               slackSend channel: '#test', color: '#439FE0', message: 'deployed in AWS', teamDomain: 'CloudzDevops', tokenCredentialId: 'slackid'
	        }
        } */
	    
    } 
}
    }
