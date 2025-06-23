# Ansible Integration in Jenkins for AWS EC2 Automation

**Author**: Fabien Andrianambinintsoa  
**DevOps Project** — Automating Docker and Docker Compose installation on AWS EC2 instances using Jenkins and Ansible.

---

## 🌐 Project Overview

This project demonstrates a complete integration of **Ansible into Jenkins**, executed from a **remote VPS control node**, to automate the setup of **Docker** and **Docker Compose** on **AWS EC2** instances.

The goal is to build a CI/CD pipeline that:
- Deploys necessary Ansible files to a **remote Ansible server (VPS)**,
- Executes an **Ansible playbook** on dynamically discovered AWS EC2 instances,
- Orchestrates everything through **Jenkins**, with full automation and security in mind.

---

## 🛠️ Technologies Used

- **Jenkins** (Pipeline as Code)
- **Ansible** (installed on remote control node)
- **AWS EC2** (targets configured via dynamic inventory)
- **Docker & Docker Compose**
- **Python 3**, **Boto3** (for AWS inventory interaction)
- **Groovy** (for Jenkins pipeline scripting)
- **Maven / Java App** (used as example application)

---

## 📁 Project Structure

devops_project/java-maven-app/
├── ansible
│ ├── ansible.cfg # Ansible configuration
│ ├── inventory_aws_ec2.yaml # AWS dynamic inventory
│ └── my-playbook.yaml # Main playbook
├── Jenkinsfile # Jenkins pipeline
├── pom.xml # Sample Java/Maven project
├── README.md # Project documentation
├── script.groovy # Optional Jenkins helper script
└── src/
└── main/
├── java/
└── resources/


---

## ⚙️ Jenkins Pipeline Workflow

### 🧩 Stage 1: Copy Files to Remote Ansible Server

```groovy
stage("copy files to ansible server") {
    steps {
        sshagent(['ansible-cred']) {
            sh 'scp -P ${SSH_PORT} -o StrictHostKeyChecking=no -r ansible/* ${SSH_TARGET}:${SSH_DEST_PATH}'
            ...
        }
    }
}

    Sends Ansible files (ansible.cfg, playbook, inventory) to a remote VPS.

    Uploads the EC2 private key for use by the playbook.

⚙️ Stage 2: Execute Playbook Remotely

stage("execute ansible playbook") {
    steps {
        sshCommand remote: remote, command: '''
            python3 -m venv ~/venv &&
            source ~/venv/bin/activate &&
            pip install --upgrade pip &&
            pip install ansible boto3 botocore &&
            ansible-playbook my-playbook.yaml
        '''
    }
}

Creates a Python virtual environment on the remote control node.

Installs required packages (ansible, boto3, botocore).

Runs the playbook to configure EC2 instances.


📦 Ansible Playbook Details
1. Install Python (via raw for non-configured EC2s)
- name: install python
  hosts: all
  tasks:
    - name: install python via raw shell
      raw: yum install -y python3

2. Install Docker
- name: install docker
  tasks:
    - name: Install docker
      yum: name=docker state=present
    - name: Add user to docker group
      user: name=ec2-user groups=docker append=yes

3. Install Docker Compose v2
- name: Install docker compose
  tasks:
    - name: Download docker compose v2
      get_url:
        url: https://github.com/docker/compose/releases/download/v2.36.2/docker-compose-linux-x86_64
        dest: /usr/local/lib/docker/cli-plugins/docker-compose
        mode: a+x
    - name: Check docker compose version
      shell: docker compose version

🌍 AWS Dynamic Inventory

File: inventory_aws_ec2.yaml
plugin: aws_ec2
regions:
  - eu-west-3
filters:
  tag:Name: web*
  instance-state-name: running
compose:
  ansible_host: public_ip_address

*Automatically discovers EC2 instances based on tags and state.
*No need for static IPs or hardcoded hostnames.

🔐 Ansible Configuration
File: ansible.cfg
[defaults]
inventory = inventory_aws_ec2.yaml
host_key_checking = false
interpreter_python = /usr/bin/python3
enable_plugins = aws_ec2
remote_user = ec2-user
private_key_file = ~/ssh-key.pem

 Prerequisites

    - AWS account with EC2 instances tagged as web-*.
    - A remote VPS with SSH access (used as Ansible control node).
    - Jenkins installed locally with required plugins:
        *SSH Agent
        *Pipeline
        *SSH Pipeline Steps

    - Jenkins credentials setup:
        ansible-cred: SSH to Ansible server
        ec2-server-key: Private key for EC2 access
        SSH_PORT, SSH_TARGET, SSH_DEST_PATH, HOST: Environment variables

🎯 Achievements

    ✅ Fully automated deployment
    ✅ Remote control using Ansible over SSH
    ✅ AWS integration with dynamic inventory
    ✅ Infrastructure as Code (IaC)
    ✅ Reproducible and version-controlled pipeline

🧑‍💻 Author

Fabien Andrianambinintsoa
DevOps & Cloud Enthusiast

    Feel free to connect with me on GitHub or LinkedIn for more discussions around DevOps, automation, and cloud deployment.

