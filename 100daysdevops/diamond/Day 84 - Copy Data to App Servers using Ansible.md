```
The Nautilus DevOps team needs to copy data from the `jump host` to all `application servers` in `Stratos DC` using Ansible. Execute the task with the following details:

  

a. Create an inventory file `/home/thor/ansible/inventory` on `jump_host` and add all application servers as managed nodes.  
  

b. Create a playbook `/home/thor/ansible/playbook.yml` on the `jump host` to copy the `/usr/src/data/index.html` file to all application servers, placing it at `/opt/data`.  
  

`Note:` Validation will run the playbook using the command `ansible-playbook -i inventory playbook.yml`. Ensure the playbook functions properly without any extra arguments.
```

Verificar los servidores
```
ssh tony@stapp01
ssh steve@stapp02
ssh banner@stapp03
```

1. Crear _inventory_
```
cd /home/thor/ansible/ && nano inventory
```

```
[stratos]
stapp01 ansible_user=tony ansible_password=Ir0nM@n ansible_ssh_common_args='-o StrictHostKeyChecking=no'
stapp02 ansible_user=steve ansible_password=Am3ric@ ansible_ssh_common_args='-o StrictHostKeyChecking=no'
stapp03 ansible_user=banner ansible_password=BigGr33n ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```

2. Crear _playbook.yml_
```
nano playbook.yml
```

```
---
- hosts: all
  become: yes
  become_user: root
  vars:
    location_jump: /usr/src/data/index.html
    location_server: /opt/data/
  tasks:
    - name: Copy index.html in all servers
      copy:
        src: "{{ location_jump }}"
        dest: "{{ location_server }}"
```

3. Ejecutar
```
ansible-playbook -i inventory playbook.yml
```
