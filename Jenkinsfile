pipeline {
    agent any
    tools {
	// Global tools to be used by the pipeline
        maven 'maven3.6' 
        jdk 'jdk8' 
    }
	
    stages {
	stage('Unit Tests') {
	    // Run Unit Tests
            steps{
                sh 'mvn -U clean test'
            }
        }

        stage("build & SonarQube analysis") {
	    agent any
            steps {
                withSonarQubeEnv('MySonarQubeServer') {
                    sh 'mvn -U clean package sonar:sonar'
                }
            }
        }

	stage("Quality Gate") {
	    steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
		
        stage ('Artifactory configuration') {
            steps {
		// specify Artifactory server
                rtServer (
                    id: "ARTIFACTORY_SERVER",
                    url: "http://172.21.0.2:8082/artifactory",
		    credentialsId: 'adminjfrog'
                )

		// specify the repositories to be used for deploying the artifacts in the Artifactory
                rtMavenDeployer (
                    id: "MAVEN_DEPLOYER",
                    serverId: "ARTIFACTORY_SERVER",
                    releaseRepo: "libs-release-local",
                    snapshotRepo: "libs-snapshot-local"
                )

    		// defines the dependencies resolution details
                rtMavenResolver (
                    id: "MAVEN_RESOLVER",
                    serverId: "ARTIFACTORY_SERVER",
                    releaseRepo: "libs-release",
                    snapshotRepo: "libs-snapshot"
                )
            }
        }

        stage ('Build & Upload Artifact') {
	    // run Maven Build and upload the built artifact to Artifactory
            steps {
                rtMavenRun (
                    tool: "maven3.6", // Tool name from Jenkins configuration
                    pom: 'pom.xml',
                    goals: 'clean install -Dmaven.test.skip=true',
                    deployerId: "MAVEN_DEPLOYER",
                    resolverId: "MAVEN_RESOLVER"
                )
            }
        }

        stage ('Publish build info') {
	    // Publish the build info in the Artifactory
            steps {
                rtPublishBuildInfo (
                    serverId: "ARTIFACTORY_SERVER"
                )
            }
        }
    }
}
