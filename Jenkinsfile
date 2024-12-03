pipeline {
    agent any
    tools {
        maven "Maven3"
        jdk "OracleJDK8"
    }
    environment {
        SNAP_REPO = "cloudops-snapshot"
        NEXUS_USER = "admin"
        NEXUS_PASS = "cloudops123"
        NEXUSIP = "172.31.94.191"
        NEXUSPORT = "8081"
        RELEASE_REPO = "cloudops-release"
        CENTRAL_REPO = "cloudops-maven-central"
	    NEXUS_GRP_REPO   = "cloudops-maven-group"
        NEXUS_CREDENTIAL_ID = "nexuslogin"
    }

    stages {
        stage("Build Stage") {
            steps {
                sh 'mvn -s settings.xml -DskipTests install'
            }
            post {
                success {
                    echo "Now Archiving..."
                    archiveArtifacts artifacts: '**/target/*.war'
                   }
            }
        }   
        stage("Unit Test") {
            steps {
                sh 'mvn -s settings.xml test'
            }
        }
        stage("CODE ANALYSIS WITH CHECKSTYLE")
            steps {
                sh 'mvn -s settings.xml checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
      }  
}
