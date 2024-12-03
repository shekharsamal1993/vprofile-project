pipeline {
    agent any
    tools {
        maven "Maven3"
        jdk "OracleJDK8"
    }
    environment {
        SNAP_REPO = "cloudops-snapshot"
        NEXUS-USER = "admin"
        NEXUS-PASS = "cloudops123"
        NEXUS_URL = "172.31.94.191"
        NEXUSPORT = "8081"
        RELEASE-REPO = "cloudops-release"
        CENTRAL-REPO = "cloudops-maven-central"
	    NEXUS_REPOGRP_ID    = "cloudops-maven-group"
        NEXUS_CREDENTIAL_ID = "nexuslogin"
    }

    stages {
        stage("Build Stage")
            steps {
                sh 'mvn -s settings.xml -DskipTests install'
        }
    }
}
