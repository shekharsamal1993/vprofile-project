def COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger',
]

pipeline {
    agent any
    environment {
        NEXUSPASS = credentials('nexuspass')
        BUILD_TIMESTAMP = sh(script: "date +%Y-%m-%d_%H-%M-%S", returnStdout: true).trim() // Add this if you need a timestamp
    }

    stages {
        stage("Setup parameters") {
            steps {
                script {
                    prperties ([
                        parameters([
                          strings(
                            defaultValue: '',
                            name: 'BUILD'
                           ),
                          strings(
                            defaultValue: '',
                            name: 'TIME'
                          )
                        ])
                    ])
                }
            }
        }
        stage("Ansible Deploy to prods"){
            steps {
               ansiblePlaybook([
               playbook: 'ansible/site.yml',
               inventory: 'ansible/prod.inventory',
               installation: 'ansible',
               colorized: true, 
               credentialsId: 'app-prod-login',
               disableHostKeyChecking: true,
               extraVars: [
                    USER: "admin",
                    PASS: "${NEXUSPASS}",
                    nexusip: "172.31.94.191",
                    reponame: "cloudops-release",
                    groupid: "QA",
                    time: "${env.BUILD_TIMESTAMP.replace(' ', '_')}",
                    build: "${env.BUILD_ID}",
                    artifactId: "cloudops",
                    cloudops_version: "cloudops-${env.BUILD_ID}-${env.BUILD_TIMESTAMP.replace(' ', '_')}.war"
                   ]
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
