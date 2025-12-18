## Session 3: What is a Playbook?

**Definition:**  
A playbook is a **YAML file** containing one or more "plays" that define tasks to be executed on managed nodes. Playbooks are **repeatable, version-controlled, and human-readable**.  

![Ansible Modules](/images/playbooks.png)

**Playbooks vs Ad-hoc Commands:**

| Feature | Ad-hoc | Playbook |
|---------|--------|----------|
| Reusability | Low | High |
| Complexity | Simple | Complex tasks |
| Version control | No | Yes, can store in Git |
| Multiple tasks | One task at a time | Multiple tasks in sequence |

---

## RECAP: YAML Basics for Playbooks

- YAML stands for **YAML Ain't Markup Language**
- **Case-sensitive**, pay attention to capitalization
- **Indentation matters** – always use spaces, not tabs
- Playbooks have **hosts, become, and tasks** sections

**Example Playbook Structure:**

```yaml
---
- name: Playbook example
  hosts: all
  become: true
  tasks:
    - name: Example task
      module_name:
        key: value
```

---

## Common Ansible Modules in Playbooks

### User Module

**Purpose:** Manage user accounts on managed nodes

```yaml
---
- name: Ansible playbook
  hosts: all
  become: true
  tasks:
    - name: Create User John
      user:
        name: john
        state: present
        shell: /bin/bash
        home: /home/john
```

**Common parameters:**
- `name` - Username
- `state` - present/absent
- `shell` - User's login shell
- `home` - Home directory path
- `groups` - Additional groups
- `create_home` - Create home directory (yes/no)

---

### File Module

**Purpose:** Manage files and directories (permissions, ownership, state)

```yaml
---
- name: Ansible playbook
  hosts: all
  become: true
  tasks:
    - name: Ensure /tmp/demo directory exists
      file:
        path: /tmp/demo
        state: directory
        mode: '0755'
```

**Common state values for file module:**
- `directory` - Create a directory
- `touch` - Create an empty file
- `absent` - Remove file/directory
- `link` - Create symbolic link
- `hard` - Create hard link

---

### YUM/DNF Modules

**Purpose:** Manage packages on RHEL-based systems

```yaml
---
- name: Ansible playbook
  hosts: all
  become: true
  tasks:
    - name: Install httpd package
      yum:
        name: httpd
        state: present
```

**Common state values for yum module:**
- `present` / `installed` - Install the package
- `absent` / `removed` - Remove the package
- `latest` - Install or update to latest version

---

### COPY Module

**Purpose:** Copy files from control node to managed nodes OR create files with content

**Method 1: Copy existing file**
```yaml
---
- name: Ansible playbook
  hosts: all
  become: true
  tasks:
    - name: Copy sample.txt to /tmp/
      copy:
        src: /home/ansible/sample.txt
        dest: /tmp/sample.txt
        mode: '0644'
```

**Method 2: Create file with inline content**
```yaml
---
- name: Ansible playbook
  hosts: all
  become: true
  tasks:
    - name: Create file with content
      copy:
        content: "Hello from Ansible"
        dest: /tmp/hello.txt
        mode: '0644'
```

---

### Key Differences: file vs copy Module

| Feature | file Module | copy Module |
|---------|-------------|-------------|
| Create empty file | ✅ `state=touch` | ❌ Must have content |
| Create file with content | ❌ | ✅ `content=` or `src=` |
| Create directory | ✅ `state=directory` | ❌ |
| Delete files | ✅ `state=absent` | ❌ |
| Set permissions | ✅ | ✅ |
| Copy from control node | ❌ | ✅ |
| Manage attributes only | ✅ | ❌ |

**When to use file:**
- Creating directories
- Creating empty files
- Deleting files/directories
- Changing permissions/ownership only
- Creating links

**When to use copy:**
- Copying files from control node
- Creating files with specific content
- Deploying configuration files
- Updating file content

---

### SERVICE Module

**Purpose:** Manage system services

```yaml
---
- name: Ansible playbook
  hosts: all
  become: true
  tasks:
    - name: Ensure httpd service is running
      service:
        name: httpd
        state: started
        enabled: true
```

**Common state values for service module:**
- `started` - Start the service if not running
- `stopped` - Stop the service if running
- `restarted` - Restart the service
- `reloaded` - Reload service configuration

**Additional parameter:**
- `enabled: true` - Service starts on boot
- `enabled: false` - Service does not start on boot

---

### LINEINFILE Module

**Purpose:** Manage individual lines in text files (add, modify, or remove lines)

**Use Case 1: Add a line if it doesn't exist**
```yaml
---
- name: Configure SSH
  hosts: all
  become: true
  tasks:
    - name: Ensure PermitRootLogin is set to no
      lineinfile:
        path: /etc/ssh/sshd_config
        regexp: '^PermitRootLogin'
        line: 'PermitRootLogin no'
        state: present
```

**Use Case 2: Add a line to a file**
```yaml
---
- name: Add line to hosts file
  hosts: all
  become: true
  tasks:
    - name: Add custom host entry
      lineinfile:
        path: /etc/hosts
        line: '192.168.1.100 myserver.local'
        state: present
```

**Use Case 3: Remove a line from a file**
```yaml
---
- name: Remove line from file
  hosts: all
  become: true
  tasks:
    - name: Remove old configuration
      lineinfile:
        path: /etc/myapp/config.conf
        regexp: '^OLD_SETTING='
        state: absent
```

**Use Case 4: Ensure a line exists after a specific line**
```yaml
---
- name: Add configuration after marker
  hosts: all
  become: true
  tasks:
    - name: Add new setting after [global] section
      lineinfile:
        path: /etc/samba/smb.conf
        insertafter: '^\[global\]'
        line: '   workgroup = MYGROUP'
```

**Common parameters:**
- `path` - File to modify
- `regexp` - Regular expression to find the line
- `line` - The line to insert/replace
- `state` - present (add/modify) or absent (remove)
- `insertafter` - Insert line after this pattern
- `insertbefore` - Insert line before this pattern
- `backrefs` - Use backreferences in line (yes/no)
- `create` - Create file if it doesn't exist (yes/no)

**Key Points:**
- If `regexp` matches, the line is replaced
- If `regexp` doesn't match and `state=present`, line is added
- Ensures only one instance of the line exists
- Idempotent - safe to run multiple times

---

### REPLACE Module

**Purpose:** Replace all instances of a pattern in a file (unlike lineinfile which handles one line)

**Use Case 1: Replace all occurrences of a word**
```yaml
---
- name: Replace text in configuration
  hosts: all
  become: true
  tasks:
    - name: Replace localhost with actual hostname
      replace:
        path: /etc/myapp/config.conf
        regexp: 'localhost'
        replace: 'server.example.com'
```

**Use Case 2: Replace a configuration value**
```yaml
---
- name: Update memory limit
  hosts: all
  become: true
  tasks:
    - name: Change memory limit to 2G
      replace:
        path: /etc/php.ini
        regexp: '^memory_limit = .*$'
        replace: 'memory_limit = 2G'
```

**Use Case 3: Comment out all matching lines**
```yaml
---
- name: Comment out old settings
  hosts: all
  become: true
  tasks:
    - name: Comment out DEBUG lines
      replace:
        path: /etc/myapp/config.conf
        regexp: '^(DEBUG=.*)$'
        replace: '# \1'
```

**Use Case 4: Replace with backreferences**
```yaml
---
- name: Update IP addresses
  hosts: all
  become: true
  tasks:
    - name: Replace old subnet with new subnet
      replace:
        path: /etc/hosts
        regexp: '^192\.168\.1\.(\d+)'
        replace: '10.0.0.\1'
```

**Common parameters:**
- `path` - File to modify
- `regexp` - Pattern to find (regular expression)
- `replace` - Text to replace with
- `backup` - Create backup before modifying (yes/no)
- `validate` - Command to validate file before saving
- `before` - Only replace matches before this pattern
- `after` - Only replace matches after this pattern

**Key Points:**
- Replaces ALL occurrences (unlike lineinfile)
- Uses Python regular expressions
- Can use backreferences with `\1`, `\2`, etc.
- Idempotent when replace text is constant

---

### Key Differences: lineinfile vs replace Module

| Feature | lineinfile Module | replace Module |
|---------|------------------|----------------|
| Scope | Single line at a time | All matching patterns |
| Use case | Add/modify/remove one line | Replace text throughout file |
| Pattern matching | Finds one line to modify | Finds all occurrences |
| Line addition | ✅ Can add new lines | ❌ Only replaces existing text |
| Line removal | ✅ Can remove lines | ❌ Cannot remove lines |
| Multiple occurrences | Handles only first match | Handles all matches |
| Backreferences | ✅ Supported | ✅ Supported |

**When to use lineinfile:**
- Adding configuration lines
- Modifying single configuration parameters
- Removing specific lines
- Ensuring specific line exists once
- Working with configuration files line by line

**When to use replace:**
- Replacing text throughout entire file
- Bulk find-and-replace operations
- Updating multiple occurrences of same pattern
- Changing values globally in file
- Text substitution operations

---

### Multiple Tasks Playbook

```yaml
---
- hosts: all
  become: true
  tasks:
    - name: Create a demo directory
      file:
        path: /tmp/demo
        state: directory
        mode: '0755'

    - name: Create folder
      file:
        path: /folder
        state: directory

    - name: Create file.txt inside /folder
      file:
        path: /folder/file.txt
        state: touch
```

---