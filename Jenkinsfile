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
                                        execCommand: 'sudo kubectl delete pods train ; sudo rm -rf /etc/containerd/train/* ; sudo yes | cp -rf /tmp/train.yaml /etc/containerd/train/ ; sudo rm -rf /tmp/*'
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
