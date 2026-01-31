# Crear directorio
```
mkdir ~/idempiere
cd idempiere
```

## Crear maquina virtual con **Vagrant**.
```
Vagrant.configure("2") do |config|  
 config.vm.box = "bento/ubuntu-22.04"  
 config.vm.network "private_network", ip: "192.168.0.30"  
  
 # Provisionamiento con Ansible  
 config.vm.provision "ansible" do |ansible|  
   ansible.playbook = "playbook/playbook.yml"  
   ansible.verbose = "v"  
 end  
end
```

# Crear **playbook**:
```
mkdir ~/idempiere/playbook
touch playbook.yml
```

```
---
- name: Instalacion de Idempiere
  hosts: all
  become: yes

  roles:
    - roles/postgres
```

## Crear roles postgres:
```
ansible-galaxy init roles/postgres
ansible-galaxy init roles/java
ansible-galaxy init roles/idempiere
ansible-galaxy init roles/install
```

### Agregar la tarea para Postgres:
```
nano roles/postgres/tasks/main.yml
```

```
- name: Instalar Postgres
  shell: |
    sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
    wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
    sudo apt-get update
    sudo apt-get -y install postgresql-15
    echo "alter user postgres password 'Idemp13r3$$'" | sudo su postgres -c "psql -U postgres"
```

### Agregar la tarea para Java:
```
nano roles/java/tasks/main.yml
```

```
- name: Instalar OpenJDK17
  shell: |
    sudo apt-get install openjdk-17-jdk-headless
```

### Agregar tarea para Idempiere
```
nano roles/idempiere/tasks/main.yml
```

```
- name: Instalar dependencias
  apt:
    name: [ "git", "wget" ]
    state: present
    update_cache: yes

- name: Descargar Installer
  shell: |
    wget "{{ url_idempiere }}"

- name: Descargar Checksum
  shell: |
    wget "{{ url_checksum }}"

- name: Verificar Checksum
  shell: |
    md5sum -c idempiereServer12Daily.gtk.linux.x86_64.zip.MD5
```

#### Agregar variables para Idempiere
```
nano roles/idempiere/vars/main.yml
```

```
url_idempiere: "https://sourceforge.net/projects/idempiere/files/v12/daily-server/idempiereServer12Daily.gtk.linux.x86_64.zip"

url_checksum: "https://sourceforge.net/projects/idempiere/files/v12/daily-server/idempiereServer12Daily.gtk.linux.x86_64.zip.MD5"
```

### Agregar tarea para Install
```
nano roles/install/tasks/main.yml
```

```

```

Inicializar **VM**:
```
vagrant up
```

Acceder a **VM**
```
vagrant ssh
```

Detener **VM**:
```
vagrant halt
```

Detener y destruir **VM**:
```
vagrant destroy
```