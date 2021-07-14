pipeline {
  agent any
  stages {
    stage ('Build') {
      steps {
        echo 'Running build automation'
        sh './gradlew build --no-daemon' // turns off background process
        echo 'Archiving Artifacts'
        archiveArtifacts artifacts: 'dist/trainSchedule.zip' // this is actually another step. archives .zip
      }
    }
    stage('Notify next Steps') {
        steps {
            echo 'The pipeline will deploy to 2 different environments'
            echo 'Deploying to Staging'
            echo 'Deploying to Production'
        }
    }
    stage('DeployToStaging') {
        when {
            branch 'master'
        }
        steps {
            withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                sshPublisher(
                    failOnError: true,
                    continueOnError: false,
                    publishers: [
                        sshPublisherDesc(
                            configName: 'staging',
                            sshCredentials: [
                                username: "$USERNAME",
                                encryptedPassphrase: "$USERPASS"
                            ],
                            transfers: [
                                sshTransfer(
                                    sourceFiles: 'dist/trainSchedule.zip',
                                    removePrefix: 'dist/',
                                    remoteDirectory: '/tmp',
                                    execCommand: 'sudo /usr/bin/systemctl stop train-schedule && rm -rf /opt/train-schedule/* && unzip /tmp/trainSchedule.zip -d /opt/train-schedule && sudo /usr/bin/systemctl start train-schedule'
                                )
                            ]
                        )
                    ]
                )
            }
        } // end of steps
    } // end of stage
    stage('DeployToProduction') {
        when {
            branch 'master'
        }
        steps {
            input 'Is the Staging environment OK?'
            milestone(1)
            withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                sshPublisher(
                    failOnError: true,
                    continueOnError: false,
                    publishers: [
                        sshPublisherDesc(
                            configName: 'production',
                            sshCredentials: [
                                username: "$USERNAME",
                                encryptedPassphrase: "$USERPASS"
                            ],
                            transfers: [
                                sshTransfer(
                                    sourceFiles: 'dist/trainSchedule.zip',
                                    removePrefix: 'dist/',
                                    remoteDirectory: '/tmp',
                                    execCommand: 'sudo /usr/bin/systemctl stop train-schedule && rm -rf /opt/train-schedule/* && unzip /tmp/trainSchedule.zip -d /opt/train-schedule && sudo /usr/bin/systemctl start train-schedule'
                                )
                            ]
                        )
                    ]
                )
            }
        }
    }
  }
}