```
The Nautilus DevOps team has some data on each app server in Stratos DC that they want to copy to a
different location. However, they want to create an archive of the data first, then they want to copy the
same to a different location on the respective app server. Additionally, there are some specific
requirements for each server. Perform the task using Ansible playbook as per requirements mentioned
below:

Create a playbook named playbook.yml under /home/thor/ansible directory on jump host, an inventory
file is already placed under /home/thor/ansible/ directory on Jump Server itself.

Create an archive blog.tar.gz (make sure archive format is tar.gz) of /usr/src/devops/ directory ( present on
each app server ) and copy it to /opt/devops/ directory on all app servers. The user and group owner of
archive blog.tar.gz should be tony for App Server 1, steve for App Server 2 and banner for App Server 3.
Note: Validation will try to run playbook using command ansible-playbook -i inventory playbook.yml so
please make sure playbook works this way, without passing any extra arguments.
```
# Ver inventario
```bash
vi /home/thor/ansible/inventory
```
### Before
```bash
stapp01 ansible_host=stapp01 ansible_ssh_pass=Ir0nM@n
ansible_user=tony
stapp02 ansible_host=stapp02 ansible_ssh_pass=Am3ric@
ansible_user=steve
stapp03 ansible_host=stapp03 ansible_ssh_pass=BigGr33n
ansible_user=banner
```

### After
```bash
stapp01 ansible_host=stapp01 ansible_ssh_pass=Ir0nM@n
ansible_user=tony ansible_ssh_common_args='-o
StrictHostKeyChecking=no'
stapp02 ansible_host=stapp02 ansible_ssh_pass=Am3ric@
ansible_user=steve ansible_ssh_common_args='-o
StrictHostKeyChecking=no'
stapp03 ansible_host=stapp03 ansible_ssh_pass=BigGr33n
ansible_user=banner ansible_ssh_common_args='-o
StrictHostKeyChecking=no'
```
# Verificar con ping
```bash
ansible all -m ping -i /home/thor/ansible/inventory
```
# Crear playbook.yml
```bash
vi /home/thor/ansible/playbook.yml
```

```yml
---
- hosts: all
  become: yes
  become_user: root
  tasks:
    - name: Comprimir fichero
      command: tar -cvzf /tmp/news.tar.gz .
      args:
        chdir: /usr/src/sysops/
        
    - name: Copiar fichero
      copy:
        src: /tmp/news.tar.gz
        dest: /opt/sysops/news.tar.gz
        owner: "{{ ansible_user }}"
        group: "{{ ansible_user }}"
        mode: '0644'
        remote_src: true
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

ls -la /opt/sysops
tar -tf /opt/sysops/news.tar.gz
```