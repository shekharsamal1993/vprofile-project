def COLOR_MAP = [
    'SUCCESS': 'good',
    'FALIURE': 'danger',
]

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
        SONARSCANNER = "sonarscanner"
        SONARSERVER  = "sonarserver"
        NEXUSPASS = credentials{'nexuspass'}
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
        stage("CODE ANALYSIS WITH CHECKSTYLE") {
            steps {
                sh 'mvn -s settings.xml checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }  
        stage('CODE ANALYSIS with SONARQUBE') {
             environment {
                scannerHome = tool("${SONARSCANNER}")
                JAVA_17_HOME = '/usr/lib/jvm/java-17-openjdk-amd64' // Adjust to your Java 17 installation path
             }
            steps {
                withSonarQubeEnv("${SONARSERVER}") {
                withEnv(["JAVA_HOME=${JAVA_17_HOME}", "PATH=${JAVA_17_HOME}/bin:${PATH}"]) {
                    sh '''
                        ${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=cloudops \
                        -Dsonar.projectName=cloudops-repo \
                        -Dsonar.projectVersion=1.0 \
                        -Dsonar.sources=src/ \
                        -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                        -Dsonar.junit.reportsPath=target/surefire-reports/ \
                        -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                        -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml
                    '''
               }
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
        stage("Publish to Nexus Repository Manager") {
             steps {
	            nexusArtifactUploader(
                nexusVersion: 'nexus3',
                protocol: 'http',
                nexusUrl: "${NEXUSIP}:${NEXUSPORT}",
                groupId: 'QA',
                version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                repository: "${RELEASE_REPO}",
                credentialsId: "${NEXUS_CREDENTIAL_ID}",
                artifacts: [
                    [artifactId: 'cloudops',
                    classifier: '',
                    file: 'target/cloudops-v2.war',
                    type: 'war']
                   ]
                )
             }
        }    
       stage("Ansible Deploy to stagings") {
           steps {
               ansiblePlaybook(
               playbook: 'ansible/site.yml',
               inventory: 'ansible/stage.inventory',
               installation: 'ansible',
               colorized: true, 
               credentialsId: 'applogin',
               disableHostKeyChecking: true,
               extraVars: [
                    USER: 'admin',
                    PASS: "${NEXUSPASS}",
                    nexusip: '172.31.94.191',
                    reponame: 'cloudops-release',
                    groupid: 'QA',
                    time: "${env.BUILD_TIMESTAMP}",
                    build: "${env.BUILD_ID}",
                    artifactId: "cloudops"
                    vprofile_version: "cloudops-${env.BUILD_ID}-${env.BUILD_TIMESTAMP}.war",

              ])
           }
       }
    }
    post {
        always {
            echo 'Slack Notificatios.'
            //Add channel name
            slackSend channel: '#jenkinscicd',
                color: COLOR_MAP[currentBuild.currentResult],
                message: "Find Status of Pipeline:- ${currentBuild.currentResult}:* job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
        }
    }    
}
