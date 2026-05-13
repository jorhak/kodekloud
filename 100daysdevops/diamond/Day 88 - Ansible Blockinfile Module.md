```
The Nautilus DevOps team wants to install and set up a simple httpd web server on all app servers in Stratos DC. Additionally, they want to deploy a sample web page for now using Ansible only. Therefore, write the required playbook to complete this task. Find more details about the task below.



We already have an inventory file under /home/thor/ansible directory on jump host. Create a playbook.yml under /home/thor/ansible directory on jump host itself.


Using the playbook, install httpd web server on all app servers. Additionally, make sure its service should up and running.


Using blockinfile Ansible module add some content in /var/www/html/index.html file. Below is the content:


Welcome to XfusionCorp!

This is  Nautilus sample file, created using Ansible!

Please do not modify this file manually!


The /var/www/html/index.html file's user and group owner should be apache on all app servers.


The /var/www/html/index.html file's permissions should be 0655 on all app servers.


Note:


i. Validation will try to run the playbook using command ansible-playbook -i inventory playbook.yml so please make sure the playbook works this way without passing any extra arguments.


ii. Do not use any custom or empty marker for blockinfile module.
```

# 1 Ver el fichero inventory
```
vi /home/thor/ansible/inventory
```

```
stapp01 ansible_host=stapp01 ansible_ssh_pass=Ir0nM@n ansible_user=tony
stapp02 ansible_host=stapp02 ansible_ssh_pass=Am3ric@ ansible_user=steve
stapp03 ansible_host=stapp03 ansible_ssh_pass=BigGr33n ansible_user=banner
```
# 2 Crear playbook.yml
```
vi /home/thor/ansible/playbook.yml
```

```
---
- hosts: all
  become: yes
  become_user: root
  
  tasks:
    - name: Install HTTPD
      yum:
        name: httpd
        state: present
        
    - name: Ensure HTTPD
      service:
        name: httpd
        state: started
        enabled: yes
        
    - name: Content index.html
      blockinfile:
        path: /var/www/html/index.html
        create: yes
        block: |
          Welcome to XfusionCorp!

          This is  Nautilus sample file, created using Ansible!

          Please do not modify this file manually!

    - name: Chance owner and permission
      file:
        path: /var/www/html/index.html
        owner: apache
        group: apache
        mode: '0655'
```

```
cd /home/thor/ansible
```

```
ansible-playbook -i inventory playbook.yml
```

# 3 Verificar
```
curl -i stapp01:80
curl -i stapp02:80
curl -i stapp03:80
```