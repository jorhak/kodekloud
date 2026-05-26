```
The Nautilus DevOps team needs to copy data from the jump host to all application servers in Stratos DC using Ansible. Execute the task with the following details:


a. Create an inventory file /home/thor/ansible/inventory on jump_host and add all application servers as managed nodes.


b. Create a playbook /home/thor/ansible/playbook.yml on the jump host to copy the /usr/src/sysops/index.html file to all application servers, placing it at /opt/sysops.


Note: Validation will run the playbook using the command ansible-playbook -i inventory playbook.yml. Ensure the playbook functions properly without any extra arguments.
```
# Crear intentory
```
vi /home/thor/ansible/inventory
```

```
stapp01 ansible_user=tony ansible_password=Ir0nM@n ansible_ssh_common_args='-o StrictHostKeyChecking=no'
stapp02 ansible_user=steve ansible_password=Am3ric@ ansible_ssh_common_args='-o StrictHostKeyChecking=no'
stapp03 ansible_user=banner ansible_password=BigGr33n ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```
# Crear playbook.yml
```
vi /home/thor/ansible/playbook.yml
```

```
---
- hosts: all
  become: yes
  become_user: root
  vars:
    origen: /usr/src/sysops/index.html
    destino: /opt/sysops
  
  tasks:
    - name: Copy of host to servers
      copy:
        src: "{{ origen }}"
        dest: "{{ destino }}"
        owner: "{{ ansible_user }}"
        group: "{{ ansible_user }}"
        mode: '0600'
```
# Ejecutar
```
cd /home/thor/ansible
ansible-playbook -i inventory playbook.yml
```
# Verificar 
```
ssh tony@stapp01
ssh steve@stapp02
ssh banner@stapp03
```

```
ls -la /opt/sysops
```