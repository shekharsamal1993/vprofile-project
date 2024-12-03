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
        NEXUS_URL = "172.31.94.191"
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
      }
    }  
}
