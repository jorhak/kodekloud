```
The Nautilus Application development team wanted to test some applications on app servers in `Stratos Datacenter`. They shared some pre-requisites with the DevOps team, and packages need to be installed on app servers. Since we are already using Ansible for automating such tasks, please perform this task using Ansible as per details mentioned below:  
  

  

1. Create an inventory file `/home/thor/playbook/inventory` on `jump host` and add all app servers in it.  
      
    
2. Create an Ansible playbook `/home/thor/playbook/playbook.yml` to install `chrony` package on `all app servers` using Ansible `yum` module.  
      
    
3. Make sure user `thor` should be able to run the playbook on `jump host`.  
    

`Note:` Validation will try to run playbook using command `ansible-playbook -i inventory playbook.yml` so please make sure playbook works this way, without passing any extra arguments.
```

# 1 Crear inventory
### Crear fichero
```
vi /home/thor/playbook/inventory
```
### Agregar todos los servidores
```
[stratos]
stapp01 ansible_user=tony ansible_password=Ir0nM@n ansible_ssh_common_args='-o StrictHostKeyChecking=no'
stapp02 ansible_user=steve ansible_password=Am3ric@ ansible_ssh_common_args='-o StrictHostKeyChecking=no'
stapp03 ansible_user=banner ansible_password=BigGr33n ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```

# 2 Crear playbook
### Crear fichero
```
vi /home/thor/playbook/playbook.yml
```
### Agregar configuracion
```
---
- hosts: stratos
  become: yes
  become_user: root
  
  tasks:
    - name: Instalar Chrony
      yum:
        name: chrony
        state: installed
```

# 3 Ejecutar
```
cd /home/thor/playbook
ansible-playbook -i inventory playbook.yml
```