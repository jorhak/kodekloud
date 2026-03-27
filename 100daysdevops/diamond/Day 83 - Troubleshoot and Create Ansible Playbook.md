```
An Ansible playbook needs completion on the jump host, where a team member left off. Below are the details:



The inventory file /home/thor/ansible/inventory requires adjustments. The playbook must run on App Server 3 in Stratos DC. Update the inventory accordingly.


Create a playbook /home/thor/ansible/playbook.yml. Include a task to create an empty file /tmp/file.txt on App Server 3.


Note: Validation will run the playbook using the command ansible-playbook -i inventory playbook.yml. Ensure the playbook works without any additional arguments.
```

1. Ingresar al servidor **App Server 3**:
	Abrimos otra terminal. Los pasos 1 y 2 solo son necesarios si no hay llegada al servidor **App Server 3**. De lo contrario nos saltamos al paso 3
```
ssh banner@stapp03
```

```
cat /etc/hosts
```

2. Agregar **IP** y **Host**:
	Volvemos a la terminal y agregamos la **IP** y **Host** del servidor **App 3**
```
sudo nano /etc/hosts
```

```
10.244.29.215  stapp03
```

3. Editar **inventory**:
```
cd /home/thor/ansible 
```

```
nano inventory
```

```
stapp03 ansible_user=banner ansible_password=BigGr33n ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```

- Verificar host
```
ansible-inventory -i inventory --list
```
- Ping
```
ansible all -m ping -i inventory
```

4. Crear **playbook.yaml**:
```
nano playbook.yml
```

```
---
- hosts: all
  become: yes
  become_user: root
  tasks:
    - name: Create file empty
      command: touch /tmp/file.txt
```

5. Ejecutar **playbook.yaml**:
```
ansible-playbook -i inventory playbook.yml
```

6. Verificar
	Para verificar abrimos otra terminal y ejecutamos:
```
ssh banner@stapp03
```

```
ls -la /tmp
```
