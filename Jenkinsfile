@Library('jenkins-pipeline-shared-libraries')_

pipeline {
    agent { label 'kie-rhel7-priority' }
    /* agent any */
    stages {
        stage('Initialize') {
            steps {
                script {
                    cleanWs()
                    }
                }
            }
        }
    }
}
