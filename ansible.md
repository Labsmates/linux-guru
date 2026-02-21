# 🤖 Ansible - Configuration Management

Guide complet d'Ansible pour automatiser la configuration et le déploiement de serveurs.

---

## 📚 Introduction

**Ansible** = Outil d'automatisation agentless (pas d'agent sur les machines cibles)

**Caractéristiques :**
- **Agentless** : Uniquement SSH requis
- **Déclaratif** : YAML (facile à lire)
- **Idempotent** : Exécutions multiples = même résultat
- **Push-based** : Control node → Managed nodes

**Use cases :**
- Configuration management
- Application deployment
- Provisioning
- Orchestration

---

## 🚀 Installation

```bash
# Debian/Ubuntu
sudo apt update
sudo apt install ansible

# RHEL/CentOS
sudo yum install ansible

# macOS
brew install ansible

# Python pip
pip3 install ansible

# Vérifier installation
ansible --version
```

---

## 📁 Structure de Projet

```
ansible-project/
├── ansible.cfg          # Configuration Ansible
├── inventory/
│   ├── production       # Inventory production
│   └── staging         # Inventory staging
├── group_vars/
│   ├── all.yml         # Variables pour tous les hosts
│   ├── webservers.yml  # Variables groupe webservers
│   └── dbservers.yml
├── host_vars/
│   └── web01.yml       # Variables host spécifique
├── roles/
│   ├── nginx/
│   │   ├── tasks/
│   │   ├── handlers/
│   │   ├── templates/
│   │   ├── files/
│   │   ├── vars/
│   │   └── defaults/
│   └── mysql/
├── playbooks/
│   ├── site.yml        # Playbook principal
│   ├── webservers.yml
│   └── dbservers.yml
├── files/              # Fichiers statiques
└── templates/          # Templates Jinja2
```

---

## 📋 Inventory

### Inventory INI Format

```ini
# inventory/production

# Hosts individuels
web01 ansible_host=192.168.1.10
web02 ansible_host=192.168.1.11

# Groupes
[webservers]
web01
web02
web03 ansible_host=192.168.1.12 ansible_port=2222

[dbservers]
db01 ansible_host=192.168.1.20
db02 ansible_host=192.168.1.21

# Groupe de groupes
[production:children]
webservers
dbservers

# Variables de groupe
[webservers:vars]
ansible_user=deploy
http_port=80
domain=example.com

[dbservers:vars]
ansible_user=dbadmin
mysql_port=3306

# Variables globales
[all:vars]
ansible_python_interpreter=/usr/bin/python3
ansible_ssh_private_key_file=~/.ssh/id_rsa
```

### Inventory YAML Format

```yaml
# inventory/production.yml
all:
  children:
    webservers:
      hosts:
        web01:
          ansible_host: 192.168.1.10
        web02:
          ansible_host: 192.168.1.11
      vars:
        ansible_user: deploy
        http_port: 80
    
    dbservers:
      hosts:
        db01:
          ansible_host: 192.168.1.20
        db02:
          ansible_host: 192.168.1.21
      vars:
        mysql_port: 3306
  
  vars:
    ansible_python_interpreter: /usr/bin/python3
```

### Dynamic Inventory (AWS)

```python
#!/usr/bin/env python3
# inventory/aws_ec2.py
import boto3
import json

ec2 = boto3.client('ec2')
response = ec2.describe_instances(
    Filters=[{'Name': 'tag:Environment', 'Values': ['production']}]
)

inventory = {'_meta': {'hostvars': {}}}
for reservation in response['Reservations']:
    for instance in reservation['Instances']:
        if instance['State']['Name'] == 'running':
            host = instance['PublicIpAddress']
            inventory.setdefault('webservers', []).append(host)

print(json.dumps(inventory))
```

---

## 🎯 Ad-Hoc Commands

```bash
# Ping tous les hosts
ansible all -i inventory/production -m ping

# Exécuter commande
ansible webservers -m shell -a "uptime"

# Copier fichier
ansible all -m copy -a "src=/tmp/file.txt dest=/etc/file.txt"

# Installer package
ansible webservers -m apt -a "name=nginx state=present" --become

# Redémarrer service
ansible webservers -m systemd -a "name=nginx state=restarted" --become

# Gather facts
ansible all -m setup

# Filtrer facts
ansible all -m setup -a "filter=ansible_distribution*"

# Parallel execution
ansible all -m ping -f 10  # 10 hosts en parallèle

# Check mode (dry-run)
ansible all -m apt -a "name=vim state=present" --check

# Verbose
ansible all -m ping -v
ansible all -m ping -vvv  # Debug
```

---

## 📘 Playbooks

### Playbook Basique

```yaml
# playbooks/webservers.yml
---
- name: Configure Web Servers
  hosts: webservers
  become: yes
  
  vars:
    http_port: 80
    domain: example.com
  
  tasks:
    - name: Install Nginx
      apt:
        name: nginx
        state: present
        update_cache: yes
    
    - name: Start Nginx
      systemd:
        name: nginx
        state: started
        enabled: yes
    
    - name: Copy nginx config
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/sites-available/{{ domain }}
      notify: Reload Nginx
    
    - name: Enable site
      file:
        src: /etc/nginx/sites-available/{{ domain }}
        dest: /etc/nginx/sites-enabled/{{ domain }}
        state: link
  
  handlers:
    - name: Reload Nginx
      systemd:
        name: nginx
        state: reloaded
```

### Exécuter Playbook

```bash
# Exécuter playbook
ansible-playbook playbooks/webservers.yml -i inventory/production

# Check mode (dry-run)
ansible-playbook playbooks/webservers.yml --check

# Limit to specific hosts
ansible-playbook playbooks/webservers.yml --limit web01

# Tags
ansible-playbook playbooks/site.yml --tags "nginx,php"
ansible-playbook playbooks/site.yml --skip-tags "slow"

# Step by step
ansible-playbook playbooks/site.yml --step

# Start at task
ansible-playbook playbooks/site.yml --start-at-task="Install MySQL"

# Extra vars
ansible-playbook playbooks/site.yml -e "env=production"
```

---

## 🔧 Modules Essentiels

### Gestion Fichiers

```yaml
# Copy file
- name: Copy file
  copy:
    src: /local/file.txt
    dest: /remote/file.txt
    owner: www-data
    group: www-data
    mode: '0644'

# Template (Jinja2)
- name: Deploy config
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  notify: Restart Nginx

# File permissions
- name: Change permissions
  file:
    path: /var/www/html
    owner: www-data
    group: www-data
    mode: '0755'
    state: directory

# Symbolic link
- name: Create symlink
  file:
    src: /path/to/source
    dest: /path/to/link
    state: link

# Remove file
- name: Remove file
  file:
    path: /tmp/oldfile
    state: absent

# Download file
- name: Download file
  get_url:
    url: https://example.com/file.zip
    dest: /tmp/file.zip
    mode: '0644'
```

### Packages

```yaml
# APT (Debian/Ubuntu)
- name: Install packages
  apt:
    name:
      - nginx
      - php-fpm
      - mysql-server
    state: present
    update_cache: yes

# YUM/DNF (RHEL/CentOS)
- name: Install packages
  yum:
    name: nginx
    state: present

# Update all packages
- name: Update all
  apt:
    upgrade: dist
    update_cache: yes

# Remove package
- name: Remove package
  apt:
    name: apache2
    state: absent
    purge: yes
```

### Services

```yaml
# Start/Stop/Restart
- name: Start service
  systemd:
    name: nginx
    state: started
    enabled: yes

- name: Restart service
  systemd:
    name: nginx
    state: restarted

# Reload (sans restart)
- name: Reload service
  systemd:
    name: nginx
    state: reloaded

# Daemon reload
- name: Reload systemd
  systemd:
    daemon_reload: yes
```

### Shell & Command

```yaml
# Shell (avec variables, pipes, redirections)
- name: Run shell command
  shell: echo $PATH > /tmp/path.txt

# Command (plus sûr, pas de shell features)
- name: Run command
  command: /usr/bin/mycommand --arg value

# Script
- name: Run script
  script: /local/scripts/deploy.sh

# Arguments
- name: Command with args
  command: /usr/bin/make install
  args:
    chdir: /opt/myapp
    creates: /opt/myapp/installed.txt  # Skip si existe
```

### Users & Groups

```yaml
# Create user
- name: Create user
  user:
    name: deploy
    shell: /bin/bash
    groups: sudo
    append: yes
    create_home: yes
    password: "{{ 'mypassword' | password_hash('sha512') }}"

# Add SSH key
- name: Add SSH key
  authorized_key:
    user: deploy
    key: "{{ lookup('file', '/local/.ssh/id_rsa.pub') }}"

# Create group
- name: Create group
  group:
    name: developers
    state: present
```

### Git

```yaml
# Clone repo
- name: Clone repository
  git:
    repo: https://github.com/user/repo.git
    dest: /opt/app
    version: main

# Update to latest
- name: Update repo
  git:
    repo: https://github.com/user/repo.git
    dest: /opt/app
    update: yes

# Specific branch/tag
- name: Checkout tag
  git:
    repo: https://github.com/user/repo.git
    dest: /opt/app
    version: v1.0.0
```

### Docker

```yaml
# Pull image
- name: Pull Docker image
  docker_image:
    name: nginx:latest
    source: pull

# Run container
- name: Run container
  docker_container:
    name: webserver
    image: nginx:latest
    state: started
    ports:
      - "80:80"
    volumes:
      - /opt/html:/usr/share/nginx/html
    env:
      ENV: production

# Docker Compose
- name: Deploy with Docker Compose
  docker_compose:
    project_src: /opt/app
    state: present
```

---

## 🎭 Roles

### Créer un Role

```bash
# Générer structure de role
ansible-galaxy init roles/nginx

# Structure créée :
roles/nginx/
├── defaults/
│   └── main.yml      # Variables par défaut
├── files/            # Fichiers statiques
├── handlers/
│   └── main.yml      # Handlers
├── meta/
│   └── main.yml      # Metadata (dépendances)
├── tasks/
│   └── main.yml      # Tasks principales
├── templates/        # Templates Jinja2
├── tests/
└── vars/
    └── main.yml      # Variables (priorité haute)
```

### Exemple de Role Nginx

```yaml
# roles/nginx/defaults/main.yml
---
nginx_port: 80
nginx_user: www-data
nginx_worker_processes: auto

# roles/nginx/tasks/main.yml
---
- name: Install Nginx
  apt:
    name: nginx
    state: present
    update_cache: yes

- name: Deploy nginx config
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  notify: Restart Nginx

- name: Ensure Nginx is started
  systemd:
    name: nginx
    state: started
    enabled: yes

# roles/nginx/handlers/main.yml
---
- name: Restart Nginx
  systemd:
    name: nginx
    state: restarted

- name: Reload Nginx
  systemd:
    name: nginx
    state: reloaded

# roles/nginx/templates/nginx.conf.j2
user {{ nginx_user }};
worker_processes {{ nginx_worker_processes }};

events {
    worker_connections 1024;
}

http {
    server {
        listen {{ nginx_port }};
        server_name {{ ansible_fqdn }};
        
        location / {
            root /var/www/html;
            index index.html;
        }
    }
}
```

### Utiliser un Role

```yaml
# playbooks/site.yml
---
- name: Configure Web Servers
  hosts: webservers
  become: yes
  
  roles:
    - nginx
    - php
    - mysql

# Avec variables
- name: Configure Web Servers
  hosts: webservers
  become: yes
  
  roles:
    - role: nginx
      nginx_port: 8080
      nginx_worker_processes: 4
```

### Ansible Galaxy (Roles publics)

```bash
# Installer role depuis Galaxy
ansible-galaxy install geerlingguy.nginx

# requirements.yml
---
- name: geerlingguy.nginx
  version: 3.1.4

- name: geerlingguy.mysql
  version: 4.3.3

# Installer depuis requirements
ansible-galaxy install -r requirements.yml

# Lister roles installés
ansible-galaxy list
```

---

## 🔀 Variables et Facts

### Variables

```yaml
# playbook vars
- hosts: all
  vars:
    http_port: 80
    domain: example.com

# vars_files
- hosts: all
  vars_files:
    - vars/common.yml
    - vars/{{ environment }}.yml

# group_vars/all.yml
---
ntp_server: pool.ntp.org
timezone: Europe/Paris

# host_vars/web01.yml
---
server_id: 1
public_ip: 203.0.113.10

# Variables d'environnement
- hosts: all
  environment:
    PATH: /usr/local/bin:{{ ansible_env.PATH }}
    HTTP_PROXY: http://proxy.example.com:8080
```

**Priorité des variables** (du plus faible au plus fort) :
```
1. role defaults
2. inventory file or script group vars
3. inventory group_vars/all
4. playbook group_vars/all
5. inventory group_vars/*
6. playbook group_vars/*
7. inventory file or script host vars
8. inventory host_vars/*
9. playbook host_vars/*
10. host facts / cached set_facts
11. play vars
12. play vars_prompt
13. play vars_files
14. role vars
15. block vars
16. task vars
17. include_vars
18. set_facts / registered vars
19. role (and include_role) params
20. include params
21. extra vars (-e)
```

### Facts

```yaml
# Gather facts automatiquement
- hosts: all
  tasks:
    - name: Display OS
      debug:
        msg: "OS is {{ ansible_distribution }} {{ ansible_distribution_version }}"

# Désactiver gather_facts
- hosts: all
  gather_facts: no

# Facts courants :
# {{ ansible_hostname }}
# {{ ansible_fqdn }}
# {{ ansible_os_family }}
# {{ ansible_distribution }}
# {{ ansible_distribution_version }}
# {{ ansible_all_ipv4_addresses }}
# {{ ansible_default_ipv4.address }}
# {{ ansible_memory_mb.real.total }}
# {{ ansible_processor_count }}

# Custom facts
- name: Set custom fact
  set_fact:
    my_custom_var: "value"
    cacheable: yes

# Facts depuis fichier
# /etc/ansible/facts.d/custom.fact (JSON)
{
  "my_app": {
    "version": "1.0.0"
  }
}

# Utilisation:
# {{ ansible_local.custom.my_app.version }}
```

---

## 🔁 Loops et Conditions

### Loops

```yaml
# Simple loop
- name: Create users
  user:
    name: "{{ item }}"
    state: present
  loop:
    - alice
    - bob
    - charlie

# Loop avec dictionnaires
- name: Create users with details
  user:
    name: "{{ item.name }}"
    groups: "{{ item.groups }}"
  loop:
    - { name: 'alice', groups: 'sudo' }
    - { name: 'bob', groups: 'developers' }

# with_items (ancien)
- name: Install packages
  apt:
    name: "{{ item }}"
    state: present
  with_items:
    - nginx
    - php-fpm
    - mysql-server

# with_dict
- name: Create directories
  file:
    path: "{{ item.value.path }}"
    state: directory
  with_dict:
    web:
      path: /var/www
    logs:
      path: /var/log/app

# with_fileglob
- name: Copy configs
  copy:
    src: "{{ item }}"
    dest: /etc/app/
  with_fileglob:
    - configs/*.conf

# Nested loops
- name: Create users on multiple servers
  user:
    name: "{{ item.0 }}"
  loop: "{{ users|product(servers)|list }}"
```

### Conditions

```yaml
# when simple
- name: Install Nginx (Debian only)
  apt:
    name: nginx
    state: present
  when: ansible_os_family == "Debian"

# Multiple conditions (AND)
- name: Install package
  apt:
    name: nginx
  when:
    - ansible_os_family == "Debian"
    - ansible_distribution_major_version == "22"

# OR condition
- name: Install package
  apt:
    name: nginx
  when: ansible_os_family == "Debian" or ansible_os_family == "RedHat"

# Variable defined
- name: Do something
  debug:
    msg: "Variable is defined"
  when: my_var is defined

# Variable not defined
- name: Do something
  debug:
    msg: "Variable is not defined"
  when: my_var is not defined

# Registered variable
- name: Check if file exists
  stat:
    path: /etc/myapp/config.yml
  register: config_file

- name: Create config if not exists
  copy:
    src: default_config.yml
    dest: /etc/myapp/config.yml
  when: not config_file.stat.exists

# Failed/Changed
- name: Run command
  command: /usr/bin/mycommand
  register: result
  ignore_errors: yes

- name: Handle failure
  debug:
    msg: "Command failed"
  when: result.failed

- name: Do something if changed
  debug:
    msg: "Something changed"
  when: result.changed
```

---

## 📊 Exemple Complet - LAMP Stack

```yaml
# playbooks/lamp.yml
---
- name: Deploy LAMP Stack
  hosts: webservers
  become: yes
  
  vars:
    mysql_root_password: "{{ vault_mysql_root_password }}"
    app_domain: example.com
    php_version: "8.1"
  
  pre_tasks:
    - name: Update apt cache
      apt:
        update_cache: yes
        cache_valid_time: 3600
  
  roles:
    - role: geerlingguy.apache
      apache_remove_default_vhost: true
      apache_vhosts:
        - servername: "{{ app_domain }}"
          documentroot: "/var/www/{{ app_domain }}"
    
    - role: geerlingguy.mysql
      mysql_root_password: "{{ mysql_root_password }}"
      mysql_databases:
        - name: appdb
      mysql_users:
        - name: appuser
          password: "{{ vault_mysql_app_password }}"
          priv: "appdb.*:ALL"
    
    - role: geerlingguy.php
      php_version: "{{ php_version }}"
      php_packages:
        - php{{ php_version }}
        - php{{ php_version }}-cli
        - php{{ php_version }}-mysql
        - php{{ php_version }}-curl
        - php{{ php_version }}-json
  
  tasks:
    - name: Create document root
      file:
        path: "/var/www/{{ app_domain }}"
        state: directory
        owner: www-data
        group: www-data
        mode: '0755'
    
    - name: Deploy application
      git:
        repo: https://github.com/user/myapp.git
        dest: "/var/www/{{ app_domain }}"
        version: main
      notify: Restart Apache
    
    - name: Install Composer
      get_url:
        url: https://getcomposer.org/installer
        dest: /tmp/composer-installer.php
      
    - name: Run Composer install
      command: php composer.phar install
      args:
        chdir: "/var/www/{{ app_domain }}"
    
    - name: Copy environment config
      template:
        src: .env.j2
        dest: "/var/www/{{ app_domain }}/.env"
        owner: www-data
        group: www-data
        mode: '0644'
  
  handlers:
    - name: Restart Apache
      systemd:
        name: apache2
        state: restarted
```

---

## 🔐 Ansible Vault - Secrets Management

```bash
# Créer vault file
ansible-vault create secrets.yml

# Éditer vault file
ansible-vault edit secrets.yml

# Encrypt existing file
ansible-vault encrypt vars/passwords.yml

# Decrypt file
ansible-vault decrypt vars/passwords.yml

# View vault file
ansible-vault view secrets.yml

# Rekey (changer password)
ansible-vault rekey secrets.yml

# Run playbook avec vault
ansible-playbook site.yml --ask-vault-pass

# Vault password file
echo "myVaultPassword123" > ~/.vault_pass
chmod 600 ~/.vault_pass
ansible-playbook site.yml --vault-password-file ~/.vault_pass

# Multiple vault passwords
ansible-playbook site.yml --vault-id dev@prompt --vault-id prod@~/.vault_prod
```

**Vault file example :**
```yaml
# secrets.yml (encrypted)
vault_mysql_root_password: SuperSecret123!
vault_mysql_app_password: AppSecret456!
vault_api_key: abc123xyz789
```

**Usage dans playbook :**
```yaml
- hosts: all
  vars_files:
    - secrets.yml
  tasks:
    - name: Use secret
      debug:
        msg: "Password is {{ vault_mysql_root_password }}"
```

---

## 🎯 Bonnes Pratiques

### 1. Idempotence

```yaml
# ✅ Bon : Idempotent
- name: Ensure Nginx is installed
  apt:
    name: nginx
    state: present

# ❌ Mauvais : Pas idempotent
- name: Install Nginx
  shell: apt-get install -y nginx
```

### 2. Check Mode Support

```yaml
- name: Create directory
  file:
    path: /opt/app
    state: directory
  check_mode: yes  # Safe pour --check
```

### 3. Tags

```yaml
- name: Install packages
  apt:
    name: nginx
  tags:
    - packages
    - nginx

# Exécuter :
# ansible-playbook site.yml --tags "nginx"
```

### 4. Error Handling

```yaml
- name: Risky operation
  command: /usr/bin/might_fail
  register: result
  ignore_errors: yes
  changed_when: result.rc == 0
  failed_when: result.rc > 2
```

---

**🎓 Prochaine étape : [Monitoring avec Prometheus & Grafana](./monitoring.md) →**
