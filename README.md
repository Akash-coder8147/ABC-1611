1. create Linux vm in azure portal with public key and ip.

public ip: 20.64.146.53
public key: Ak-vm-practice_key.pem


2.store the certificate in ansible vault.

ansible_user: azureuser
ansible_host: localhost
ansible_connection: local
ansible_ssh_private_key_file: /home/azureuser/.ssh/Ak-vm-practice_key.pem

execution command: ansible-vault create secrets.yml
cat secrets.yml


 3. Use Ansible Playbook to Perform Tasks on the VM

mkdir -p ~/ansible-tf-task
cd ~/ansible-tf-task


[azure-vm]
20.64.146.53 ansible_user=azureuser ansible_ssh_private_key_file=/home/azureuser/Ak-vm-practice_key.pem

create a file : palybook.yml

- name: Automate Terraform Tasks on Local VM
  hosts: azure-vm
  become: true
  vars_files:
    - secrets.yml

  tasks:
    - name: Install dependencies
      apt:
        name: ['git', 'wget', 'unzip', 'curl']
        state: present
        update_cache: true

    - name: Download and install Terraform
      shell: |
        curl -fsSL https://releases.hashicorp.com/terraform/1.7.5/terraform_1.7.5_linux_amd64.zip -o terraform.zip
        unzip terraform.zip
        mv terraform /usr/local/bin/
        terraform -version
      args:
        chdir: /tmp

    - name: Clone Git repo with Terraform scripts
      git:
        repo: https://github.com/Akash-coder8147/your-tf-repo.git
        dest: /home/azureuser/tf-repo

    - name: Run Terraform scripts
      shell: |
        terraform init
        terraform apply -auto-approve
      args:
        chdir: /home/azureuser/tf-repo
      register: tf_output

    - name: Show Terraform output
      debug:
        var: tf_output.stdout_lines

    - name: Cleanup cloned repo
      file:
        path: /home/azureuser/tf-repo
        state: absent


ansible-playbook -i inventory.ini playbook.yml --ask-vault-pass



 4. Create GitHub Repo named ABC-1611

     url:https://github.com/Akash-coder8147/ABC-1611
    

6. Push Ansible Configuration to GitHub

   mkdir ABC-1611 && cd ABC-1611
cp ../playbook.yml ../inventory.ini ../secrets.yml .
git init
git remote add origin https://github.com/youruser/ABC-1611.git
git add .
git commit -m "Added Ansible setup for VM automation"
git push -u origin master
