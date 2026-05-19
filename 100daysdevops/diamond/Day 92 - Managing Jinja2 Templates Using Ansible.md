```
One of the Nautilus DevOps team members is working on to develop a role for `httpd` installation and configuration. Work is almost completed, however there is a requirement to add a jinja2 template for `index.html` file. Additionally, the relevant task needs to be added inside the role. The inventory file `~/ansible/inventory` is already present on `jump host` that can be used. Complete the task as per details mentioned below:

  

a. Update `~/ansible/playbook.yml` playbook to run the `httpd` role on `App Server 1`.  
  

b. Create a jinja2 template `index.html.j2` under `/home/thor/ansible/role/httpd/templates/` directory and add a line `This file was created using Ansible on <respective server>` (for example `This file was created using Ansible on stapp01` in case of `App Server 1`). Also please make sure not to hard code the server name inside the template. Instead, use `inventory_hostname` variable to fetch the correct value.  
  

c. Add a task inside `/home/thor/ansible/role/httpd/tasks/main.yml` to copy this template on `App Server 1` under `/var/www/html/index.html`. Also make sure that `/var/www/html/index.html` file's permissions are `0644`.  
  

d. The user/group owner of `/var/www/html/index.html` file must be respective sudo user of the server (for example `tony` in case of `stapp01`).  
  

`Note:` Validation will try to run the playbook using command `ansible-playbook -i inventory playbook.yml` so please make sure the playbook works this way without passing any extra arguments.
```
# 1 Ver inventory
```
cat /home/thor/ansible/inventory
```

```
stapp01 ansible_host=stapp01 ansible_ssh_pass=Ir0nM@n ansible_user=tony
stapp02 ansible_host=stapp02 ansible_ssh_pass=Am3ric@ ansible_user=steve
stapp03 ansible_host=stapp03 ansible_ssh_pass=BigGr33n ansible_user=banner
```
# Modificar playbook.yml
```
vi /home/thor/ansible/playbook.yml
```

```
---
- hosts: stapp02
  become: yes
  become_user: root
  roles:
    - role/httpd
```
En hosts se le agrego el nombre del servidor al cual se le van hacer los cambios.
# 3 Crear template jinja2
```
vi /home/thor/ansible/role/httpd/templates/index.html.j2
```

```
This file was created using Ansible on {{ inventory_hostname }}
```
# 4 Crear variable
```
vi /home/thor/ansible/role/httpd/vars/main.yml
```

```
inventory_hostname: "stapp02"
```
Al igual que en **playbook.yml** tengo que cambiar el servidor correspondiente.
# 5 Copiar fichero
```
vi /home/thor/ansible/role/httpd/tasks/main.yml
```

```
---
# tasks file for role/test

- name: install the latest version of HTTPD
  yum:
    name: httpd
    state: latest

- name: Start service httpd
  service:
    name: httpd
    state: started
      
- name: Copy index.html.j2
  template:
    src: /home/thor/ansible/role/httpd/templates/index.html.j2
    dest: /var/www/html/index.html
    owner: "{{ ansible_env.SUDO_USER }}"
    group: "{{ ansible_env.SUDO_USER }}"
    mode: '0644'
```
En **src** podemos colocar la ruta completa o simplemente:
```
   src: index.html.j2
```
# Ejecutar
```
cd /home/thor/ansible
ansible-playbook -i inventory playbook.yml
```
