# 🛠️ GitHub Actions + Ansible: Install NGINX on Ubuntu Linux

This repository demonstrates how to automate the installation of **NGINX** on a remote **Ubuntu Linux** server using **GitHub Actions** and **Ansible**.

---

## 🧩 Overview

We’ll set up a GitHub Actions workflow that:
1. Uses Ansible to connect to a remote Ubuntu host.
2. Runs a playbook to install and start the NGINX service.

---

## 📝 Prerequisites

- A remote Ubuntu Linux server (tested on Ubuntu 20.04+).
- SSH access to the server (with a private key).
- A GitHub repository.
- A basic understanding of Ansible and GitHub Actions.

---

## 🔑 SSH Key Setup

1. **Generate SSH key named `ansible_key`** on your local machine:

   ```bash
   ssh-keygen -t rsa -b 4096 -f ~/.ssh/ansible_key
This creates:

~/.ssh/ansible_key (private key)

~/.ssh/ansible_key.pub (public key)

Copy the public key to your Ubuntu server:

ssh-copy-id -i ~/.ssh/ansible_key.pub <user>@<host>
Replace <user> with your SSH username (e.g., ubuntu) and <host> with your server's IP or domain.

🔐 Setup Secrets in GitHub
Go to your repository > Settings > Secrets and variables > Actions > New repository secret:

Name	Description
SSH_PRIVATE_KEY	Content of your ansible_key (private key)
HOST	IP address or domain of the server
USER	SSH username (e.g., ubuntu)
To copy the private key to your clipboard:

bash
Copy
Edit
cat ~/.ssh/ansible_key
Then paste it into the SSH_PRIVATE_KEY secret.

📁 Repository Structure
bash
Copy
Edit
.
├── .github
│   └── workflows
│       └── deploy-nginx.yml     # GitHub Actions workflow
├── ansible
│   ├── hosts                    # Inventory file
│   └── install-nginx.yml       # Ansible playbook
├── README.md
📦 Ansible Playbook
ansible/install-nginx.yml:

yaml
- name: Install and start NGINX
  hosts: webservers
  become: yes

  tasks:
    - name: Install NGINX
      apt:
        name: nginx
        state: present
        update_cache: yes

    - name: Ensure NGINX is running
      service:
        name: nginx
        state: started
        enabled: yes

🗂️ Inventory File
[webservers]
{{ HOST }}
The {{ HOST }} placeholder will be dynamically replaced in the workflow.

🤖 GitHub Actions Workflow
.github/workflows/deploy-nginx.yml:

yaml
Copy
Edit
name: Deploy NGINX via Ansible

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v3

      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.x'

      - name: Install Ansible
        run: |
          python -m pip install --upgrade pip
          pip install ansible

      - name: Add SSH key
        run: |
          mkdir -p ~/.ssh
          echo "${{ secrets.SSH_PRIVATE_KEY }}" > ~/.ssh/id_rsa
          chmod 600 ~/.ssh/id_rsa

      - name: Run Ansible Playbook
        env:
          ANSIBLE_HOST_KEY_CHECKING: "False"
        run: |
          sed -i "s/{{ HOST }}/${{ secrets.HOST }}/g" ansible/hosts
          ansible-playbook -i ansible/hosts ansible/install-nginx.yml -u ${{ secrets.USER }}

✅ Test It Out
Push your code to the main branch.

Go to Actions tab.

Watch the workflow run and deploy NGINX.

🔍 Verify
SSH into your server and run:

curl -I http://localhost
You should see an HTTP 200 response from NGINX.

🧼 Cleanup
To remove NGINX:

sudo apt remove nginx -y
sudo apt autoremove -y

📄 License
MIT License


