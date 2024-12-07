def COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger',
]

pipeline {
    agent any
    environment {
        NEXUSPASS = credentials('nexuspass')
    }

    stages {
        stage("Setup parameters") {
            steps {
                script {
                    properties([
                        parameters([
                            string(
                                defaultValue: '',
                                name: 'BUILD'
                            ),
                            string(
                                defaultValue: '',
                                name: 'TIME'
                            )
                        ])
                    ])
                }
            }
        }
        stage("Ansible Deploy to prods") {
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
                        time: "${env.TIME}",
                        build: "${env.BUILD}",
                        artifactId: "cloudops",
                        cloudops_version: "cloudops-${env.BUILD}-${env.TIME}.war"
                    ]
                ])
            }
        }
    }

    post {
        always {
            echo 'Slack Notification.'
            // Add channel name
            slackSend channel: '#jenkinscicd',
                      color: COLOR_MAP[currentBuild.currentResult],
                      message: "Find Status of Pipeline:- ${currentBuild.currentResult}:* job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
        }
    }
}
