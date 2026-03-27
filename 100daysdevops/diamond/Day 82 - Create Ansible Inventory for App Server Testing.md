```
The Nautilus DevOps team is testing Ansible playbooks on various servers within their stack. They've placed some playbooks under /home/thor/playbook/ directory on the jump host and now intend to test them on app server 3 in Stratos DC. However, an inventory file needs creation for Ansible to connect to the respective app. Here are the requirements:


a. Create an ini type Ansible inventory file /home/thor/playbook/inventory on jump host.


b. Include App Server 3 in this inventory along with necessary variables for proper functionality.


c. Ensure the inventory hostname corresponds to the server name as per the wiki, for example stapp01 for app server 1 in Stratos DC.


Note: Validation will execute the playbook using the command ansible-playbook -i inventory playbook.yml. Ensure the playbook functions properly without any extra arguments.
```

playbook.yml
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

Ingresar por ssh a stapp03
```
ssh banner@stapp03
cat /etc/hosts
```

Abrimos otra terminal capturamos su IP y Host de stapp03. Y lo copiamos en /etc/hosts de JUMP-HOST
```
nano /etc/hosts
```

```
# Kubernetes-managed hosts file.
127.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
fe00::0 ip6-mcastprefix
fe00::1 ip6-allnodes
fe00::2 ip6-allrouters
10.244.97.152   jump-host

# Entries added by HostAliases.
10.0.15.5       docker-registry-mirror.kodekloud.com
10.244.49.93    stapp03
```

1. Ir a directorio
```
cd /home/thor/playbook/
```

2. Crear inventario
```
nano inventory
```

```
[server]
stapp03 ansible_host=10.244.49.93 ansible_user=banner ansible_password=BigGr33n
```

3. Verificar inventario
```
ansible-inventory -i inventory --list
```

4. Ping a server
```
ansible server -m ping -i inventory
```

5. Ejecutar playbook
```
ansible-playbook -i inventory playbook.yml
```
