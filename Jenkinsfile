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
        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    app = docker.build('dangcongphung/train-schedule')
                    app.inside {
                        sh 'echo $(curl localhost:8080)'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'DockerHub_dangcongphung_credential') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('DeployToProduction using docker') {
            when {
                branch 'master'
            }
            steps {
                input 'Deploy to Production using docker ?'
                milestone(1)
                withCredentials([usernamePassword(credentialsId: 'root_AP29_AP31_credential', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    script {
                        sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@172.20.4.107 \"docker pull dangcongphung/train-schedule:${env.BUILD_NUMBER}\""
                        try {
                            sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@172.20.4.107 \"docker stop train-schedule\""
                            sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@172.20.4.107 \"docker rm train-schedule\""
                        } catch (err) {
                            echo: 'caught error: $err'
                        }
                        sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@172.20.4.107 \"docker run --restart always --name train-schedule -p 8080:8080 -d dangcongphung/train-schedule:${env.BUILD_NUMBER}\""
                    }
                }
            }
        }
       stage('DeployToProduction1') {
            when {
                branch 'master'
            }
            steps {
                input 'Does the staging environment look OK 1'
                milestone(1)
                withCredentials([usernamePassword(credentialsId: 'root_AP29_AP31_credential', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    sshPublisher(
                        failOnError: true,
                        continueOnError: false,
                        publishers: [
                            sshPublisherDesc(
                                configName: 'Pro_UATK8SMN01',
                                sshCredentials: [
                                    username: "$USERNAME",
                                    encryptedPassphrase: "$USERPASS"
                                ], 
                                transfers: [
                                    sshTransfer(
                                        sourceFiles: 'train.yaml',
                                        remoteDirectory: '/tmp',
                                        execCommand: 'sudo date >> /tmp/data.txt sudo kubectl delete pods train && rm -rf /etc/containerd/train/*'
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
