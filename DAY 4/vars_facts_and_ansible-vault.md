# Session 4: Ansible Variables, Facts, and Vault

---

## 1. Ansible Variables

### Simple vars Example

```yaml
---
- name: Install package with variable
  hosts: webservers
  vars:
    package_name: httpd
  
  tasks:
    - name: Install package
      apt:
        name: "{{ package_name }}"
        state: present
```

### List Variables

```yaml
---
- name: Install multiple packages
  hosts: all
  vars:
    packages:
      - git
      - vim
      - curl
  
  tasks:
    - name: Install packages
      yum:
        name: "{{ item }}"
        state: present
      loop: "{{ packages }}"
```

---

## 2. Using vars_files

### Directory Structure
```
project/
├── playbook.yml
└── vars/
    └── users.yml
```

### vars/users.yml
```yaml
---
linux_user: test
```

### playbook.yml
```yaml
---
- name: Setup web server
  hosts: webservers
  vars_files:
    - vars/users.yml
  
  tasks:
    - name: Create user
      user:
        name: "{{ linux_user }}"
        state: present
```

---

## 3. Ansible Facts

Ansible automatically gathers **facts** about the remote systems before running tasks.  
These facts are variables containing information about the system, such as:

- OS type and version  
- IP addresses  
- Hostname  
- Memory and CPU information  
- Mounted filesystems

### Using Facts in Playbook

```yaml
---
- name: Install based on OS
  hosts: all
  
  tasks:
    - name: Install on Debian
      apt:
        name: apache2
        state: present
      when: ansible_os_family == "Debian"
    
    - name: Install on RedHat
      yum:
        name: httpd
        state: present
      when: ansible_os_family == "RedHat"
```

### Common Facts
```yaml
ansible_hostname          # server name
ansible_os_family         # Debian, RedHat, etc
ansible_distribution      # Ubuntu, CentOS, etc
ansible_default_ipv4.address  # IP address
ansible_memtotal_mb       # Total memory
ansible_processor_vcpus   # CPU count
```

### View Facts Command
```bash
ansible localhost -m setup
ansible all -m setup -a "filter=ansible_os_*"
```

---

## 5. Ansible Vault

### Create Encrypted File
```bash
ansible-vault create secrets.yml
```

**Enter password, then add:**
```yaml
---
password: "MySecret123"
```

### Common Vault Commands
```bash
ansible-vault create secrets.yml    # Create new
ansible-vault edit secrets.yml      # Edit existing
ansible-vault view secrets.yml      # View content
ansible-vault encrypt file.yml      # Encrypt existing
ansible-vault decrypt file.yml      # Decrypt
ansible-vault rekey secrets.yml     # Change password
```

### Use Encrypted File in Playbook

**playbook.yml:**
```yaml
---
- name: Use secrets
  hosts: all
  vars_files:
    - secrets.yml
  
  tasks:
    - name: Create user John
      user:
        name: John
        password: "{{ password | password_hash('sha512') }}"
        state: present
```

**Run playbook:**
```bash
ansible-playbook playbook.yml --ask-vault-pass
```

### Use Password File
```bash
echo "MyVaultPassword" > vault_pass
ansible-playbook playbook.yml --vault-password-file=vault_pass
```

---

## Best Practices

- Use `vars_files` for better organization
- Always encrypt sensitive data with vault
- Disable facts if not needed (faster execution)
- Use meaningful variable names
- Never commit passwords to git

---