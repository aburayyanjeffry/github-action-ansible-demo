![banner](images/github-action-ansible.png)
# 🛠️ GitHub Actions + Ansible: Install NGINX on Ubuntu Linux

This repository demonstrates how to automate the installation of **NGINX** on a remote **Ubuntu Linux** server using **GitHub Actions** and **Ansible**.

---

## 🧩 Overview

We’ll set up a GitHub Actions workflow that:
1. Uses Ansible to connect to a remote Ubuntu host.
2. Runs a playbook to install and start the NGINX service.

---

## 📝 Prerequisites

- A remote Ubuntu Linux server with a known IP address or hostname. This IP or hostname will be referred to as `TARGET_HOST` from here onward.
- SSH access to the server (with a private key).
- A GitHub repository.
- A basic understanding of Ansible and GitHub Actions.

---

## 🔑 SSH Key Setup

1. **Generate SSH key named `ansible_key`** on your local machine:

   ```bash
   ssh-keygen -t rsa -b 4096 -f ansible_key
   ```
This creates:
```
ansible_key (private key)
ansible_key.pub (public key)
```

Copy the public key to your Ubuntu server:
```
ssh-copy-id -i ~/.ssh/ansible_key.pub ubuntu@HOST
```

## 🔐 Setup Secrets in GitHub
Go to your repository > Settings > Secrets and variables > Actions > New repository secret:

Name	Description
SSH_PRIVATE_KEY	Content of your ansible_key (private key)
HOST	IP address or domain of the server

![repo](images/github-repo-vars.png)
## 📁 Repository Structure
GitHub Actions requires a specific directory structure for its workflows. The structure should be as follows.
For better organization, we’ll place the Ansible playbook in a directory named `Ansible`. 
```
.
├── .github
│   └── workflows
│       └── deploy-nginx.yml     # GitHub Actions workflow
├── ansible
│   └── install-nginx.yml       # Ansible playbook
├── README.md

```

## 📦 Ansible Playbook
ansible/install-nginx-playbook.yml:

```yaml
- name: Install Hello World NGINX
  hosts: webservers
  become: yes

  tasks:
    - name: Install NGINX
      apt:
        name: nginx
        state: present
        update_cache: yes

    - name: Create Hello World index.html
      copy:
        dest: /var/www/html/index.html
        content: |
          <html>
          <head><title>Hello World</title></head>
          <body><h1>Hello, World from Ansible!</h1></body>
          </html>

    - name: Ensure NGINX is running
      service:
        name: nginx
        state: started
        enabled: yes
```


## 🤖 GitHub Actions Workflow
This will create Invetory file from the HOST variable and apply the playbook

.github/workflows/deploy-nginx.yml:
```yaml
name: Install Nginx Playbook

on:
  push:
    branches:
    - main
  workflow_dispatch:
    # allows manual trigger from the Actions tab

jobs:
  run-ansible:
    runs-on: ubuntu-latest

    env:
      TARGET_HOST: ${{ secrets.TARGET_HOST }}

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Install Ansible
      run: |
        sudo apt update
        sudo apt install -y ansible

    - name: Set up SSH key
      run: |
        mkdir -p ~/.ssh
        echo "${{ secrets.SSH_PRIVATE_KEY }}" > ~/.ssh/id_rsa
        chmod 600 ~/.ssh/id_rsa
        ssh-keyscan -H "$TARGET_HOST" >> ~/.ssh/known_hosts
      shell: bash

    - name: Create dynamic inventory
      run: |
        echo "[webservers]" > hosts.ini
        echo "$TARGET_HOST ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/id_rsa" >> hosts.ini

    - name: Test manual SSH connection (debug)
      run: |
        ssh -o StrictHostKeyChecking=no -i ~/.ssh/id_rsa ubuntu@$TARGET_HOST "echo 'SSH Success 🎉'"

    - name: Run Ansible Playbook
      run: |
        ansible-playbook ansible/nginx_hello.yml -i hosts.ini

    - name: Verify Nginx Hello World Page
      run: |
        echo "Checking if Nginx is serving the Hello World page..."
        response=$(curl -s http://$TARGET_HOST)
        echo "$response"

        if echo "$response" | grep -qi "hello world"; then
          echo "✅ Nginx is serving the expected content!"
        else
          echo "❌ Nginx did not serve the expected Hello World content."
          exit 1
        fi
```

## ✅ Test It Out
Push your code to the main branch.

Go to Actions tab.

Watch the workflow run and deploy NGINX.
![log](images/github-action-logs.png)

## 🔍 Verify
We already verified it in the workflow by curling the TARGET_HOST. You can double-check it by accessing it through your web browser.
![log](images/github-browser .png)

## 🧼 Cleanup
To remove NGINX if it is not needed at the server:
```bash
sudo apt remove nginx -y
sudo apt autoremove -y
```

## 📄 License
MIT License


