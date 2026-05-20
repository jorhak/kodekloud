```
The Nautilus DevOps team had a discussion about, how they can train different team members to use Ansible for different automation tasks. There are numerous ways to perform a particular task using Ansible, but we want to utilize each aspect that Ansible offers. The team wants to utilise Ansible's conditionals to perform the following task:  
  

An `inventory` file is already placed under `/home/thor/ansible` directory on `jump host`, with all the `Stratos DC app servers` included.  
  

Create a playbook `/home/thor/ansible/playbook.yml` and make sure to use Ansible's `when` conditionals statements to perform the below given tasks.

  

1. Copy `blog.txt` file present under `/usr/src/data` directory on `jump host` to `App Server 1` under `/opt/data` directory. Its user and group owner must be user `tony` and its permissions must be `0744` .  
      
    
2. Copy `story.txt` file present under `/usr/src/data` directory on `jump host` to `App Server 2` under `/opt/data` directory. Its user and group owner must be user `steve` and its permissions must be `0744` .  
      
    
3. Copy `media.txt` file present under `/usr/src/data` directory on `jump host` to `App Server 3` under `/opt/data` directory. Its user and group owner must be user `banner` and its permissions must be `0744`.  
      
    

`NOTE:` You can use `ansible_nodename` variable from gathered facts with `when` condition. Additionally, please make sure you are running the play for all hosts i.e use `- hosts: all`.  
  

`Note:` Validation will try to run the playbook using command `ansible-playbook -i inventory playbook.yml`, so please make sure the playbook works this way without passing any extra arguments.
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
# 2 Crear playbook.yml
```
vi /home/thor/ansible/playbook.yml
```

```
---
- hosts: all
  become: true
  become_user: root
  gather_facts: true
  vars:
    source: /usr/src/data
    destination: /opt/data
    permission: '0744'
  
  tasks:
    - name: Copy file in App Server 1
      copy:
        src: "{{ source }}/blog.txt"
        dest: "{{ destination }}/blog.txt"
        owner: "{{ ansible_user }}"
        group: "{{ ansible_user }}"
        mode: "{{ permission }}"
      when: ansible_nodename == "stapp01"
      
    - name: Copy file in App Server 2
      copy:
        src: "{{ source }}/story.txt"
        dest: "{{ destination }}/story.txt"
        owner: "{{ ansible_user }}"
        group: "{{ ansible_user }}"
        mode: "{{ permission }}"
      when: ansible_nodename == "stapp02"
      
    - name: Copy file in App Server 3
      copy:
        src: "{{ source }}/media.txt"
        dest: "{{ destination }}/media.txt"
        owner: "{{ ansible_user }}"
        group: "{{ ansible_user }}"
        mode: "{{ permission }}"
      when: ansible_nodename == "stapp03"
```
# Ejecutar
```
cd /home/thor/ansible
ansible-playbook -i inventory playbook.yml
```
# Verificar
```
ssh tony@stapp01
ls -la /opt/data
```

```
ssh steve@stapp02
ls -la /opt/data
```

```
ssh banner@stapp03
ls -la /opt/data
```