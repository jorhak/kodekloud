```
An Ansible playbook needs completion on the jump host, where a team member left off. Below are the details:



The inventory file /home/thor/ansible/inventory requires adjustments. The playbook must run on App Server 3 in Stratos DC. Update the inventory accordingly.


Create a playbook /home/thor/ansible/playbook.yml. Include a task to create an empty file /tmp/file.txt on App Server 3.


Note: Validation will run the playbook using the command ansible-playbook -i inventory playbook.yml. Ensure the playbook works without any additional arguments.
```
# Editar inventory
```
vi /home/thor/ansible/inventory
```

```
stapp02 ansible_user=steve ansible_password=Am3ric@ ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```
# Crear playbook.yml
```
vi /home/thor/ansible/playbook.yml
```

```
---
- hosts: stapp02
  become: yes
  become_user: root
  vars:
    dest_server: /tmp/file.txt
  
  tasks:
    - name: Create file empty
      file:
        path: "{{ dest_server }}"
        owner: "{{ ansible_user }}"
        group: "{{ ansible_user }}"
        mode: '0644'
        state: touch
```
# Ejecutar
```
cd /home/thor/ansible
ansible-playbook -i inventory playbook.yml
```