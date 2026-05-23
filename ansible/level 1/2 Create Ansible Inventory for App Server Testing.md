```
The Nautilus DevOps team is testing Ansible playbooks on various servers within their stack. They've placed some playbooks under /home/thor/playbook/ directory on the jump host and now intend to test them on app server 2 in Stratos DC. However, an inventory file needs creation for Ansible to connect to the respective app. Here are the requirements:


a. Create an ini type Ansible inventory file /home/thor/playbook/inventory on jump host.


b. Include App Server 2 in this inventory along with necessary variables for proper functionality.


c. Ensure the inventory hostname corresponds to the server name as per the wiki, for example stapp01 for app server 1 in Stratos DC.


Note: Validation will execute the playbook using the command ansible-playbook -i inventory playbook.yml. Ensure the playbook functions properly without any extra arguments.
```
# Crear inventory
```
vi /home/thor/playbook/inventory
```

```
stapp02 ansible_user=steve ansible_password=Am3ric@ ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```
# Ver playbook.yml
```
vi /home/thor/playbook/playbook.yml
```

```
---
- hosts: all
  become: yes
  become_user: root
  tasks:
    - name: Install httpd package
      yum:
        name: httpd
        state: installed

    - name: Start service httpd
      service:
        name: httpd
        state: started
```
# Ejecutar
```
cd /home/thor/playbook
ansible-playbook -i inventory playbook.yml
```
# Verificar
```
curl -I --connect-timeout 5 stapp02
```