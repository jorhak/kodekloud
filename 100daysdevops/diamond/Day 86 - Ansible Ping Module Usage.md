```
The Nautilus DevOps team is planning to test several Ansible playbooks on different app servers in Stratos DC. Before that, some pre-requisites must be met. Essentially, the team needs to set up a password-less SSH connection between Ansible controller and Ansible managed nodes. One of the tickets is assigned to you; please complete the task as per details mentioned below:


a. Jump host is our Ansible controller, and we are going to run Ansible playbooks through thor user from jump host.


b. There is an inventory file /home/thor/ansible/inventory on jump host. Using that inventory file test Ansible ping from jump host to App Server 1, make sure ping works.
```

1. Crear llave
```
ssh-keygen -t rsa -b 4096
```

2. Copiar llave publica en servidor
```
ssh-copy-id -i ~/.ssh/id_rsa.pub banner@stapp03
```

3. Veriricar conexion
```
ssh banner@stapp03
```

4. Modificar _inventory_
```
nano /home/thor/ansible/inventory
```

```
stapp03 ansible_user=banner ansible_ssh_private_key_file=~/.ssh/id_rsa
```

2. Verificar llegada
```
ansible all -m ping -i /home/thor/ansible/inventory
```

