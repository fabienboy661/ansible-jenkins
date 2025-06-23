pipeline {
    agent any
    environment {
        SSH_PORT       = credentials('ssh-port')
        SSH_TARGET     = credentials('ssh-target')
        SSH_DEST_PATH  = credentials('ssh-dest-path')
        HOST  = credentials('host')
    }
    stages {
        stage("copy files to ansible server") {
            steps {
                script {
                    echo "copying all necessary files to ansible control node"
                    sshagent(['ansible-cred']){
                        sh '''
                            scp -P ${SSH_PORT} -o StrictHostKeyChecking=no -r ansible/* ${SSH_TARGET}:${SSH_DEST_PATH}
                        '''
                        withCredentials([sshUserPrivateKey(credentialsId: 'ec2-server-key', keyFileVariable: 'keyfile', usernameVariable: 'user')]) {
                            sh '''
                                scp -P ${SSH_PORT} -o StrictHostKeyChecking=no ${keyfile} ${SSH_TARGET}:${SSH_DEST_PATH}/ssh-key.pem
                            '''
                        }
                    }
                }
            }
        }
        stage("execute ansible playbook")   {
            steps {
                script {
                    echo "calling ansible playbook to configure ec2 instances"
                    def remote = [:]
                    remote.name = "ansible-server"
                    remote.host = HOST
                    // remote.port = SSH_PORT
                    remote.port = Integer.parseInt(SSH_PORT)
                    remote.allowAnyHosts = true

                    withCredentials([sshUserPrivateKey(credentialsId: 'ansible-cred', keyFileVariable: 'keyfile', usernameVariable: 'user')]) {
                        remote.user = user
                        remote.identityFile = keyfile
                        sshCommand remote: remote, command: '''
                            python3 -m venv ~/venv &&
                            source ~/venv/bin/activate &&
                            pip install --upgrade pip &&
                            pip install ansible boto3 botocore &&
                            source ~/venv/bin/activate &&
                            ansible-playbook my-playbook.yaml
                        '''
                    }
                }
            }
        }
    }   
}