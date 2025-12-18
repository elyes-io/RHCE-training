# Session 2: Ad-hoc Commands

![Ansible Modules](/images/modules.png)

## Objective
By the end of this session, you will be able to:

1. Use Ansible ad-hoc commands to perform basic tasks
2. Understand and use different types of modules
3. Understand idempotency and how Ansible ensures consistent results

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
