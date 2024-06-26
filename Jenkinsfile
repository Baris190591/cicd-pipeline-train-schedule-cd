pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('DeployToStaging') {
            when {
                branch 'master'
            }
            steps {
                sshPublisher(
                    failOnError: true,
                    continueOnError: false,
                    publishers: [
                        sshPublisherDesc(
                            configName: 'staging-ssh',
                            transfers: [
                                sshTransfer(
                                    sourceFiles: 'dist/trainSchedule.zip',
                                    removePrefix: 'dist/',
                                    remoteDirectory: '/tmp',
                                    execCommand: 'sudo /usr/bin/systemctl stop train-schedule; echo "Stopped service"; rm -rf /opt/train-schedule/*; echo "Cleared directory"; unzip /tmp/trainSchedule.zip -d /opt/train-schedule; echo "Unzip completed"; sudo /usr/bin/systemctl start train-schedule; echo "Started service"'
                                )
                            ]
                        )
                    ]
                )
            }
        }
        stage('DeployToProduction') {
            when {
                branch 'master'
            }
            steps {
                input 'Does the staging environment look OK?'
                milestone(1)
                sshPublisher(
                    failOnError: true,
                    continueOnError: false,
                    publishers: [
                        sshPublisherDesc(
                            configName: 'production-ssh',
                            transfers: [
                                sshTransfer(
                                    sourceFiles: 'dist/trainSchedule.zip',
                                    removePrefix: 'dist/',
                                    remoteDirectory: '/tmp',
                                    execCommand: 'sudo /usr/bin/systemctl stop train-schedule; echo "Stopped service"; rm -rf /opt/train-schedule/*; echo "Cleared directory"; unzip /tmp/trainSchedule.zip -d /opt/train-schedule; echo "Unzip completed"; sudo /usr/bin/systemctl start train-schedule; echo "Started service"'
                                )
                            ]
                        )
                    ]
                )
            }
        }
    }
}
