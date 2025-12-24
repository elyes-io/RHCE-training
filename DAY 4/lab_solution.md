# Lab Solutions: Ansible Variables, Facts, and Vault

---

## Exercise 1: Working with Simple Variables

### Task 1.1: Create a Playbook with Inline Variables

**File: `install_package.yml`**

```yaml
---
- name: Install package with variable
  hosts: all
  vars:
    package_name: vim
  
  tasks:
    - name: Install vim on RedHat family
      yum:
        name: "{{ package_name }}"
        state: present
      when: ansible_os_family == "RedHat"
```

**Run the playbook:**
```bash
ansible-playbook install_package.yml
```

---

## Exercise 2: Using List Variables and Loops

### Task 2.1: Install Multiple Packages

**File: `install_multiple.yml`**

```yaml
---
- name: Install multiple packages
  hosts: all
  vars:
    packages:
      - git
      - wget
      - curl
      - httpd
  
  tasks:
    - name: Install packages
      yum:
        name: "{{ item }}"
        state: present
      loop: "{{ packages }}"
```

**Run the playbook:**
```bash
ansible-playbook install_multiple.yml
```

---

## Exercise 3: External Variable Files

### Task 3.1: Create Variable Files

**Directory structure:**
```bash
mkdir -p project/vars
cd project
```

**File: `vars/users.yml`**

```yaml
---
app_user: webadmin
app_group: webadmin
app_shell: /bin/bash
```

### Task 3.2: Use Variable Files in Playbook

**File: `create.yml`**

```yaml
---
- name: Create user from variable file
  hosts: all
  vars_files:
    - vars/users.yml
  
  tasks:
    - name: Create application group
      group:
        name: "{{ app_group }}"
        state: present
    
    - name: Create application user
      user:
        name: "{{ app_user }}"
        group: "{{ app_group }}"
        shell: "{{ app_shell }}"
        create_home: yes
        state: present
```

**Run the playbook:**
```bash
ansible-playbook create.yml
```

---

## Exercise 4: Exploring Ansible Facts

### Task 4.1: View All Facts

**View all facts for all hosts:**
```bash
ansible all -m setup
```

**Filter OS-related information:**
```bash
ansible all -m setup -a "filter=ansible_os_*"
```

**Filter IPv4 information:**
```bash
ansible all -m setup -a "filter=ansible_default_ipv4"
```

**Filter memory information:**
```bash
ansible all -m setup -a "filter=ansible_mem*"
```

### Task 4.2: Create Playbook Using Facts

**File: `system_info.yml`**

```yaml
---
- name: Collect system information
  hosts: all
  become: yes
  
  tasks:
    - name: Create system specs file
      copy:
        content: |
          Hostname: {{ ansible_hostname }}
          OS Family: {{ ansible_os_family }}
          Distribution: {{ ansible_distribution }} {{ ansible_distribution_version }}
          IP Address: {{ ansible_default_ipv4.address }}
          Total Memory: {{ ansible_memtotal_mb }} MB
          CPU Cores: {{ ansible_processor_vcpus }}
        dest: /etc/specs.txt
        mode: '0644'
```

**Run the playbook:**
```bash
ansible-playbook system_info.yml
```

**Verify the file:**
```bash
ansible all -m command -a "cat /etc/specs.txt" --become
```

### Task 4.3: Conditional Execution Based on Facts

**File: `conditional_install.yml`**

```yaml
---
- name: Conditional installation based on facts
  hosts: all
  
  tasks:
    - name: Install Apache on RedHat family
      yum:
        name: httpd
        state: present
      when: ansible_os_family == "RedHat"
    
    - name: Show message if enough memory
      debug:
        msg: "System has enough memory (>= 2GB)"
      when: ansible_memtotal_mb >= 2048
    
    - name: Warn if low memory
      debug:
        msg: "WARNING: System has less than 2GB memory"
      when: ansible_memtotal_mb < 2048
```

**Run the playbook:**
```bash
ansible-playbook conditional_install.yml
```

---

## Exercise 5: Ansible Vault - Securing Sensitive Data

### Task 5.1: Create Encrypted File

**Create encrypted file:**
```bash
ansible-vault create secrets.yml
```

**Enter vault password when prompted (e.g., `MyVaultPass123`)**

**File content: `secrets.yml`**
```yaml
---
db_password: "SuperSecret123!"
api_key: "abc123def456ghi789"
admin_password: "AdminPass456"
```

**Save and exit (`:wq` in vim)**

### Task 5.2: View and Edit Encrypted Files

**Try to cat the vault file:**
```bash
cat secrets.yml
```

**Output explanation:**
The file shows encrypted content starting with `$ANSIBLE_VAULT;1.1;AES256` followed by encrypted data. This is because Ansible Vault uses AES256 encryption to secure the file contents.

**View encrypted content:**
```bash
ansible-vault view secrets.yml
```

**Edit encrypted file:**
```bash
ansible-vault edit secrets.yml
```

**Add a new variable:**
```yaml
---
db_password: "SuperSecret123!"
api_key: "abc123def456ghi789"
admin_password: "AdminPass456"
ssh_key: "ssh-rsa-key-here"
```

**View again to confirm:**
```bash
ansible-vault view secrets.yml
```

### Task 5.3: Encrypt Existing File

**Create plain file: `database.yml`**
```bash
cat > database.yml << EOF
---
db_host: localhost
db_port: 5432
db_name: myapp
db_user: dbadmin
db_password: "PlainTextPassword"
EOF
```

**Encrypt the file:**
```bash
ansible-vault encrypt database.yml
```

**Verify you cannot read it:**
```bash
cat database.yml
```

**Output:** You'll see encrypted content instead of plain text.

### Task 5.4: Use Vault in Playbook

**File: `deploy_app.yml`**

```yaml
---
- name: Deploy application with secrets
  hosts: all
  vars_files:
    - secrets.yml
  
  tasks:
    - name: Create admin user
      user:
        name: admin
        password: "{{ admin_password | password_hash('sha512') }}"
        state: present
```

**Run with vault password prompt:**
```bash
ansible-playbook deploy_app.yml --ask-vault-pass
```

**Create password file:**
```bash
echo "MyVaultPass123" > .vault_pass
```

**Run with password file:**
```bash
ansible-playbook deploy_app.yml --vault-password-file=.vault_pass
```

---

## Verification Commands

**Check if user was created:**
```bash
ansible all -m command -a "id admin" --become
```

**Check if packages are installed:**
```bash
ansible all -m command -a "rpm -q vim git wget curl httpd"
```

**Check specs file:**
```bash
ansible all -m command -a "cat /etc/specs.txt" --become
```

---

## Additional Vault Commands

**Decrypt a file:**
```bash
ansible-vault decrypt database.yml
```

**Change vault password (rekey):**
```bash
ansible-vault rekey secrets.yml
```

**Re-encrypt:**
```bash
ansible-vault encrypt database.yml
```

---