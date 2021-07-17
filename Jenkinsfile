pipeline {
  environment {
     DOCKER_HUB_USERNAME = 'aleon1220'
  }
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
    stage('Build Docker Image') {
        when {
            branch 'deploy/containerised-app'
        }
        steps {
            script {
                app = docker.build("aleon1220/train-schedule")
                app.inside {
                    sh 'echo $(curl localhost:8080)'
                }
            }
        }
    }
    stage('Push Docker Image') {
        when {
            branch 'deploy/containerised-app'
        }
        steps {
            script {
                docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                    app.push("${env.BUILD_NUMBER}")
                    app.push("latest")
                }
            }
        }
    }
    stage('Notify next Steps') {
        steps {
            echo 'The pipeline will deploy to Staging and Production'
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
            branch 'deploy/*'
        }
        steps {
            input 'Is the Staging environment OK?'
            milestone(1)
            withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                script {
                sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@${env.prod_ip} \"docker pull ${env.DOCKER_HUB_USERNAME}/train-schedule:${env.BUILD_NUMBER}\""
                try {
                   sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@${env.prod_ip} \"docker stop train-schedule\""
                   sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@${env.prod_ip} ${env.DOCKER_HUB_USERNAME}\"docker rm train-schedule\""
                } catch (err) {
                    echo: 'caught error: $err'
                }
                sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@${env.prod_ip} \"docker run --restart always --name train-schedule -p 8080:8080 -d {env.DOCKER_HUB_USERNAME}/train-schedule:${env.BUILD_NUMBER}\""
                }
            }
        }
    }
  }
}