```
The Nautilus DevOps team wants to install and set up a simple httpd web server on all app servers in Stratos DC. Additionally, they want to deploy a sample web page for now using Ansible only. Therefore, write the required playbook to complete this task. Find more details about the task below.



We already have an inventory file under /home/thor/ansible directory on jump host. Create a playbook.yml under /home/thor/ansible directory on jump host itself.


Using the playbook, install httpd web server on all app servers. Additionally, make sure its service should up and running.


Using blockinfile Ansible module add some content in /var/www/html/index.html file. Below is the content:


Welcome to XfusionCorp!

This is  Nautilus sample file, created using Ansible!

Please do not modify this file manually!


The /var/www/html/index.html file's user and group owner should be apache on all app servers.


The /var/www/html/index.html file's permissions should be 0744 on all app servers.


Note:


i. Validation will try to run the playbook using command ansible-playbook -i inventory playbook.yml so please make sure the playbook works this way without passing any extra arguments.


ii. Do not use any custom or empty marker for blockinfile module.
```
# Ver inventory
```bash
vi /home/thor/ansible/inventory
```
### Before
```bash
stapp01 ansible_host=stapp01 ansible_ssh_pass=Ir0nM@n ansible_user=tony 
stapp02 ansible_host=stapp02 ansible_ssh_pass=Am3ric@ ansible_user=steve
stapp03 ansible_host=stapp03 ansible_ssh_pass=BigGr33n ansible_user=banner
```
### After
```bash
[stratos]
stapp01 ansible_host=stapp01 ansible_ssh_pass=Ir0nM@n ansible_user=tony ansible_ssh_common_args='-o StrictHostKeyChecking=no'
stapp02 ansible_host=stapp02 ansible_ssh_pass=Am3ric@ ansible_user=steve ansible_ssh_common_args='-o StrictHostKeyChecking=no'
stapp03 ansible_host=stapp03 ansible_ssh_pass=BigGr33n ansible_user=banner ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```
### Verficar conexion con ping
```bash
ansible all -m ping -i /home/thor/ansible/inventory
```
# Crear playbook.yml
```bash
vi /home/thor/ansible/playbook.yml
```

```yml
---
- hosts: stratos
  become: yes
  
  tasks:
    - name: Actualizar repositorio
      dnf: 
        update_cache: yes
        
    - name: Instalar httpd web server
      dnf:
        name: httpd
        state: present
    
    - name: 
      blockinfile:
        create: true
        path: "/var/www/html/index.html"
        owner: "apache"
        group: "apache"
        mode: '0744'
        block: |
          Welcome to XfusionCorp!
          This is  Nautilus sample file, created using Ansible!
          Please do not modify this file manually!

    - name: Make sure a service unit is running
      systemd_service:
        state: started
        name: httpd
```
## Ejecutar
```bash
cd /home/thor/ansible
ansible-playbook -i inventory playbook.yml
```
# Verificar
```bash
ssh tony@stapp01
ssh steve@stapp02
ssh banner@stapp03
curl localhost
```

- '/var/www/html/index.html' file having wrong permissions on stapp01