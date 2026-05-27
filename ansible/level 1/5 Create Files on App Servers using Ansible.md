```
The Nautilus DevOps team is testing various Ansible modules on servers in Stratos DC. They're currently focusing on file creation on remote hosts using Ansible. Here are the details:


a. Create an inventory file ~/playbook/inventory on jump host and include all app servers.


b. Create a playbook ~/playbook/playbook.yml to create a blank file /home/nfsdata.txt on all app servers.


c. Set the permissions of the /home/nfsdata.txt file to 0777.


d. Ensure the user/group owner of the /home/nfsdata.txt file is tony on app server 1, steve on app server 2 and banner on app server 3.


Note: Validation will execute the playbook using the command ansible-playbook -i inventory playbook.yml, so ensure the playbook functions correctly without any additional arguments.
```
# 1 Crear inventory
```
vi /home/thor/playbook/inventory
```

```
stapp01 ansible_user=tony ansible_password=Ir0nM@n ansible_ssh_common_args='-o StrictHostKeyChecking=no'
stapp02 ansible_user=steve ansible_password=Am3ric@ ansible_ssh_common_args='-o StrictHostKeyChecking=no'
stapp03 ansible_user=banner ansible_password=BigGr33n ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```
# 2 Crear playbook.yml
```
vi /home/thor/playbook/playbook.yml
```

```
---
- hosts: all
  become: yes
  become_user: root
  vars:
    empty_file: /home/nfsdata.txt
    
  tasks:
    - name: Create file empty
      file:
        path: "{{ empty_file }}"
        owner: "{{ ansible_user }}"
        group: "{{ ansible_user }}"
        state: touch
        mode: '0777'   
```
# 3 Ejecutar
```
cd /home/thor/playbook
ansible-playbook -i inventory playbook.yml
```
# Verificar
```
ssh tony@stapp01
ssh steve@stapp02
ssh banner@stapp03
```

```
ls -la /home
```