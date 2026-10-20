```
One of the DevOps team members has created a zip archive on jump host in Stratos DC that needs to be extracted and copied over to all app servers in Stratos DC itself. Because this is a routine task, the Nautilus DevOps team has suggested automating it. We can use Ansible since we have been using it for other automation tasks. Below you can find more details about the task:


We have an inventory file under /home/thor/ansible directory on jump host, which should have all the app servers added already.


There is a zip archive /usr/src/security/devops.zip on jump host.


Create a playbook.yml under /home/thor/ansible/ directory on jump host itself to perform the below given tasks.


Unzip /usr/src/security/devops.zip archive in /opt/security/ location on all app servers.


Make sure the extracted data must has the respective sudo user as their user and group owner, i.e tony for app server 1, steve for app server 2, banner for app server 3.


The extracted data permissions must be 0744.


Note: Validation will try to run the playbook using command ansible-playbook -i inventory playbook.yml so please make sure playbook works this way, without passing any extra arguments.
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
## Verificar con ping
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
  become_user: root
  
  tasks:
    - name: Descomprimir fichero en servidor
      unarchive:
        src: /usr/src/security/devops.zip
        dest: /opt/security/
        owner: "{{ ansible_user }}"
        group: "{{ ansible_user }}"
        mode: '0744'
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

ls -la /opt/security && ls -la /opt/security/unarchive
```