# Session 5: Jinja2 Templates, Firewalld, and Ansible Collections

---

## 1. Jinja2 and Template

### Introduction

In this section, we will explore how to use `jinja2` syntax and the `template` module.

### Jinja2 Syntax and Template Module

Jinja2 is a template engine used in Ansible to generate files dynamically. It allows you to incorporate variables, loops, and conditions into configuration files using files with the `.j2` extension.

### Basic Jinja2 Syntax:

**Variables**: Used to insert dynamic values.

```jinja2
{{ variable_name }}
```

**Loops**: Used to repeat a section of code.

```jinja2
{% for item in list %}
{{ item }}
{% endfor %}
```

**Conditions**: Used to insert conditional sections of code.

```jinja2
{% if condition %}
Some content
{% endif %}
```

### Template Module:

The Ansible `template` module uses Jinja2 to create files based on templates. It allows you to copy a template file to a location on the remote host, replacing variables with their current values.

**Example of template module:**

```yaml
- name: Render a configuration file from template
  template:
    src: /path/to/template.j2
    dest: /path/to/destination/file
```

### Complete Example

**Project structure:**
```
project/
├── playbook.yml
└── templates/
    └── config.txt.j2
```

**File: `templates/config.j2`**

```jinja2
{% for host in groups.all %}
{{ hostvars[host].ansible_default_ipv4.address }} {{ hostvars[host].ansible_fqdn }} {{ hostvars[host].ansible_hostname }}
{% endfor %}
```

**File: `playbook.yml`**

```yaml
---
- name: Deploy configuration file
  hosts: all
  tasks:
    - name: Create config file from template
      template:
        src: templates/config.j2
        dest: /tmp/config.txt
```

---

## 2. Ansible Collections

### What is a Collection?

Ansible collections are a distribution format that can include:
- Modules
- Plugins
- Roles
- Playbooks

Format: `namespace.collection_name`

### Installing Collections

**Install a collection:**

```bash
ansible-galaxy collection install ansible.posix
```

**Install from a requirements file:**

**File: `requirements.yml`**

```yaml
---
collections:
  - name: ansible.posix
    version: "1.5.4"
```

```bash
ansible-galaxy collection install -r requirements.yml
```

---

## 3. Configuring collections_path in ansible.cfg

### What is collections_path?

The `collections_path` parameter in `ansible.cfg` specifies where Ansible looks for collections.

**Default locations:**
- `~/.ansible/collections`
- `/usr/share/ansible/collections`

### Creating ansible.cfg

**File: `ansible.cfg`**

```ini
[defaults]
inventory = inventory.ini
remote_user = ansible
# Collections path - Ansible searches in order
collections_path = ./collections:~/.ansible/collections:/usr/share/ansible/collections

[privilege_escalation]
become = True
```

### Structure with Custom Collections

```
project/
├── ansible.cfg
├── collections/
│   └── ansible_collections/
│       └── ansible/
│           └── posix/
├── inventory.ini
└── playbooks/
```

**Install to custom path:**

```bash
ansible-galaxy collection install ansible.posix -p ./collections
```

**File: `ansible.cfg`**

```ini
[defaults]
inventory = inventory.ini
remote_user = ansible
# Collections path - Ansible searches in order
collections_path = ./collections:~/.ansible/collections:/usr/share/ansible/collections:/home/server/project/collections

[privilege_escalation]
become = True
```

---

## 4. Module ansible.posix.firewalld

### What is firewalld?

Firewalld is a dynamic firewall manager for Linux that uses zones and services to manage network traffic rules.


### Firewalld Module Syntax

```yaml
- name: Manage firewall rules
  ansible.posix.firewalld:
    service: http           # Service name
    port: 8080/tcp         # Or specific port
    permanent: yes         # Make rule persistent
    state: enabled         # enabled or disabled
    immediate: yes         # Apply immediately
```

### Example: Allow HTTP/HTTPS

```yaml
---
- name: Configure firewall for web server
  hosts: webservers
  become: yes
  
  tasks:
    - name: Install firewalld
      yum:
        name: firewalld
        state: present
    
    - name: Start and enable firewalld
      service:
        name: firewalld
        state: started
        enabled: yes
    
    - name: Allow HTTP traffic
      ansible.posix.firewalld:
        service: http
        permanent: yes
        state: enabled
        immediate: yes
    
    - name: Allow HTTPS traffic
      ansible.posix.firewalld:
        service: https
        permanent: yes
        state: enabled
        immediate: yes
```

### Example: Allow Custom Ports

```yaml
---
- name: Allow custom application ports
  hosts: appservers
  become: yes
  tasks:
    - name: Allow application port 8080
      ansible.posix.firewalld:
        port: 8080/tcp
        permanent: yes
        state: enabled
        immediate: yes
    
    - name: Allow database port 5432
      ansible.posix.firewalld:
        port: 5432/tcp
        permanent: yes
        state: enabled
        immediate: yes
```

### Example: Remove Firewall Rules

```yaml
---
- name: Remove firewall rules
  hosts: all
  become: yes
  tasks:
    - name: Remove HTTP service
      ansible.posix.firewalld:
        service: http
        permanent: yes
        state: disabled
        immediate: yes
    - name: Remove custom port
      ansible.posix.firewalld:
        port: 8080/tcp
        permanent: yes
        state: disabled
        immediate: yes
```

---

## Best Practices

- Keep templates in a `templates/` directory
- Use `.j2` extension for Jinja2 templates
- Always specify collection versions in `requirements.yml`
- Use FQCN (Fully Qualified Collection Names) in playbooks
- Make firewall rules permanent with `permanent: yes`
- Use `immediate: yes` to apply firewall changes without reload
- Keep `ansible.cfg` in project root
- Document required collections in project README

---