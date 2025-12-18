# Session 2: Ansible Modules & Ad-hoc Commands

![Ansible Modules](/images/modules.png)

## Objective
By the end of this session, you will be able to:

1. Use Ansible ad-hoc commands to perform basic tasks
2. Understand and use different types of modules
3. Write and run simple tasks using ad-hoc commands
4. Understand idempotency and how Ansible ensures consistent results

---

## Ad-hoc Commands

**Definition:**  
Ad-hoc commands are single commands run from the command line without writing a playbook.

## Understanding Modules

**Definition:** Modules are reusable scripts that perform tasks on managed nodes.

**Types of Modules:**

| Type | Description |
|------|-------------|
| Core Modules | Shipped with Ansible (file, yum, service, command, copy) |
| Extras | Contributed by community |

**Demo Modules:**
- file – manage files/directories
- user – manage user accounts
- yum / apt – install/remove packages
- service – start/stop/enable services
- copy – copy files to managed nodes

---

**Usage:**  
```bash
ansible <host-pattern> -m <module> -a "<module-arguments>"
```
### Common Examples

# Check uptime
```bash
ansible all -m command -a "uptime"
``` 

# Check disk space
```bash
ansible all -m command -a "df -h"
```

# Manage services
```bash
ansible webservers -m service -a "name=httpd state=started"

```

# Copy files
```bash
ansible webservers -m copy -a "src=/tmp/index.html dest=/var/www/html/index.html"
```

# Install packages
# ansible all -m yum -a "name=httpd state=present"   # RHEL/CentOS

---

## Characteristics

**Definition:** Running the same task multiple times produces the same result.

**Example:**  
```bash
ansible all -m file -a "path=/tmp/demo state=directory mode=0755"
``` 
- Directory will exist with the correct permissions regardless of how many times the command is run.

---

## Using Variables in Ad-hoc Commands

# Display the current user
```bash
ansible all -m shell -a "echo $USER"
```

```bash
ansible all -m shell-a "echo $HOME"
```

---

## What is a Playbook?

**Definition:**  
A playbook is a **YAML file** containing one or more “plays” that define tasks to be executed on managed nodes. Playbooks are **repeatable, version-controlled, and human-readable**.  

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

**User Module** 
```yaml
---
- name: Ansible playbook
  hosts: all
  become: true
  tasks:
    - name: Create User John
      user:
        name: John
```

**File Module** 
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

**YUM/DNF Modules**
```yaml
---
- name: Ansible playbook
  hosts: all
  become: true
  tasks:
    - name: Install vim editor
      yum:
        name: httpd
        state: present
```

**COPY Module**
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

**SEVICE Module**

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

**Multiple Tasks Playbook**

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

# Understanding the `state` Attribute in Ansible (General Concept)

In Ansible, the `state` attribute is one of the most commonly used parameters in modules. It allows you to declare the **desired condition** of a resource on a managed node.

Think of `state` as a **goal**: it tells Ansible:

> "I want this resource to be in this specific state."

Ansible will check the current condition of the resource and **only make changes if the current state does not match the desired state**, ensuring **idempotence** (task dosen't get executed more than one time UNCHANGED STATE).

---

## Key Points About `state`

### 1. Represents Desired State
- `state` defines what the resource **should be**, not how to make it that way.
- Example: a file can have a state of `present` (it should exist) or `absent` (it should be removed).

### 2. Module-Specific
- Different modules support different `state` values.
- Always consult the module documentation to know which states are valid.

### 3. Supports Idempotency
- Ansible compares the **current state** with the **desired state**.
- If the resource is already in the desired state, Ansible does nothing.
- This prevents unnecessary changes and ensures predictable automation.

### 4. Examples of Typical `state` Goals
- **Existence**: Does the resource exist? (`present` / `absent`)
- **Running Status**: Is a service running or stopped? (`started` / `stopped`)
- **Configuration**: Is a directory or file created with proper permissions? (`directory` / `file`)
