```
There are some files that need to be created on all app servers in `Stratos DC`. The Nautilus DevOps team want these files to be owned by user `root` only however, they also want that the app specific user to have a set of permissions on these files. All tasks must be done using Ansible only, so they need to create a playbook. Below you can find more information about the task.  
  

  

Create a playbook named `playbook.yml` under `/home/thor/ansible` directory on `jump host`, an `inventory` file is already present under `/home/thor/ansible` directory on `Jump Server` itself.  
  

1. Create an empty file `blog.txt` under `/opt/security/` directory on app server 1. Set some `acl` properties for this file. Using `acl` provide `read '(r)'` permissions to `group tony` (i.e `entity` is `tony` and `etype` is `group`).  
      
    
2. Create an empty file `story.txt` under `/opt/security/` directory on app server 2. Set some `acl` properties for this file. Using `acl` provide `read + write '(rw)'` permissions to `user steve` (i.e `entity` is `steve` and `etype` is `user`).  
      
    
3. Create an empty file `media.txt` under `/opt/security/` on app server 3. Set some `acl` properties for this file. Using `acl` provide `read + write '(rw)'` permissions to `group banner` (i.e `entity` is `banner` and `etype` is `group`).  
      
    

`Note:` Validation will try to run the playbook using command `ansible-playbook -i inventory playbook.yml` so please make sure the playbook works this way, without passing any extra arguments.
```
# 1 Agregar variables en inventory
```
vi /home/thor/ansible/inventory
```

```
stapp01 ansible_host=stapp01 ansible_ssh_pass=Ir0nM@n ansible_user=tony server_entity=tony server_etype=group server_perms=r server_path="/opt/finance" server_file=/opt/finance/blog.txt
stapp02 ansible_host=stapp02 ansible_ssh_pass=Am3ric@ ansible_user=steve server_entity=steve server_etype=user server_perms=rw server_path="/opt/finance" server_file=/opt/finance/story.txt
stapp03 ansible_host=stapp03 ansible_ssh_pass=BigGr33n ansible_user=banner server_entity=banner server_etype=group server_perms=rw server_path="/opt/finance" server_file=/opt/finance/media.txt
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
    - name: Create directory
      file:
        path: "{{ server_path }}"
        state: directory
        mode: '0755'
    
    - name: Create file
      file:
        path: "{{ server_file }}"
        state: touch
        mode: '0644'
      
    - name: Apply ACL
      acl:
        path: "{{ server_file }}"
        entity: "{{ server_entity }}"
        etype: "{{ server_etype }}"
        permissions: "{{ server_perms }}"
        state: present
      
```
# 3 Ejecutar
```
cd /home/thor/ansible
ansible-playbook -i inventory playbook.yml
```

# Verificar los permisos
Debemos ingresar a los servidores y ejecutamos:
```
getfacl /opt/finance/blog.txt
getfacl /opt/finance/story.txt
getfacl /opt/finance/media.txt
```