```
The Nautilus Application development team wanted to test some applications on app servers in Stratos Datacenter. They shared some pre-requisites with the DevOps team, and packages need to be installed on app servers. Since we are already using Ansible for automating such tasks, please perform this task using Ansible as per details mentioned below:


Create an inventory file /home/thor/playbook/inventory on jump host and add all app servers in it.


Create an Ansible playbook /home/thor/playbook/playbook.yml to install chrony package on all  app servers using Ansible yum module.


Make sure user thor should be able to run the playbook on jump host.


Note: Validation will try to run playbook using command ansible-playbook -i inventory playbook.yml so please make sure playbook works this way, without passing any extra arguments.
```
# Crear inventory
```bash
vi /home/thor/playbook/inventory
```

```bash
[stratos]
stapp01 ansible_user=tony ansible_password=Ir0nM@n ansible_ssh_common_args='-o StrictHostKeyChecking=no'
stapp02 ansible_user=steve ansible_password=Am3ric@ ansible_ssh_common_args='-o StrictHostKeyChecking=no'
stapp03 ansible_user=banner ansible_password=BigGr33n ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```
## Verificar conexion
```bash
ansible all -m ping -i /home/thor/playbook/inventory
```
# Crear playbook.yml
```bash
vi /home/thor/playbook/playbook.yml
```

```yml
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
## Ejecutar playbook.yml
```bash
cd /home/thor/playbook
ansible-playbook -i inventory playbook.yml
```
# Verificar
```bash
ssh tony@stapp01
ssh steve@stapp02
ssh banner@stapp03

chronyc -h
```