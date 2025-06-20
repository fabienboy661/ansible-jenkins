pipeline {
    agent any
    environment {
        SSH_PORT = '34879'
        SSH_TARGET = 'mokolos1@92.113.25.202'
        SSH_DEST_PATH = '/home/mokolos1'
    }
    stages {
        stage("copy files to ansible server") {
            steps {
                script {
                    echo "copying all necessary files to ansible control node"
                    sshanget(['ansible-cred']){
                        sh "scp -P ${SSH_PORT} -o StrictHostKeyChecking=no ansible/* ${SSH_TARGET}:${SSH_DEST_PATH}"

                        withCredentials([sshUserPrivateKey(credentialsId: 'ec2-server-key', keyFileVariable: 'keyfile', usernamaVariabel: 'user')]) {
                            sh "scp ${keyfile} ${SSH_TARGET}:${SSH_DEST_PATH}/ssh-key.pem"
                        }
                    }
                }
            }
        }
    }   
}