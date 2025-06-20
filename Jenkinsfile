pipeline {
    agent any
    environment {
        SSH_PORT       = credentials('ssh-port')  
        SSH_TARGET     = credentials('ssh-target') 
        SSH_DEST_PATH  = credentials('ssh-dest-path') 
    }
    stages {
        stage("copy files to ansible server") {
            steps {
                script {
                    echo "copying all necessary files to ansible control node"
                    sshagent(['ansible-cred']){
                        sh "scp -P ${SSH_PORT} -o StrictHostKeyChecking=no -r ansible/* ${SSH_TARGET}:${SSH_DEST_PATH}"
                        withCredentials([sshUserPrivateKey(credentialsId: 'ec2-server-key', keyFileVariable: 'keyfile', usernameVariable: 'user')]) {
                            sh "scp ${keyfile} ${SSH_TARGET}:${SSH_DEST_PATH}/ssh-key.pem"
                        }
                    }
                }
            }
        }
    }   
}