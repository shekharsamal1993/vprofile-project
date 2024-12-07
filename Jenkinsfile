def COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger',
    'UNSTABLE': 'warning'
]

pipeline {
    agent any
    environment {
        NEXUSPASS = credentials('nexuspass')
    }

    parameters {
        string(name: 'BUILD', defaultValue: '1.0', description: 'Build version')
        string(name: 'TIME', defaultValue: '2024-12-01_12:00', description: 'Build timestamp')
    }

    stages {
        stage("Setup parameters") {
            steps {
                script {
                    properties([
                        parameters([
                            string(defaultValue: '', name: 'BUILD'),
                            string(defaultValue: '', name: 'TIME')
                        ])
                    ])
                }
            }
        }
        stage("Ansible Deploy to prod") {
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
                        time: "${params.TIME}",  // Correct usage of params
                        build: "${params.BUILD}",
                        artifactId: "cloudops",
                        cloudops_version: "cloudops-${params.BUILD}-${params.TIME.replace(' ', '%20').replace(':', '%3A')}.war"
                    ]
                ])
            }
        }
    }

    post {
        always {
            echo 'Slack Notification.'
            slackSend channel: '#jenkinscicd',
                      color: COLOR_MAP[currentBuild.currentResult],
                      message: "Find Status of Pipeline:- ${currentBuild.currentResult}:* job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
        }
    }
}
