# Day 2 Lab Exercises - Solutions

## Table of Contents
- [Exercise 1: Writing Your First Playbook](#exercise-1-writing-your-first-playbook)
- [Exercise 2: HTTPD Installation](#exercise-2-httpd-installation)
- [Exercise 3: User Management Playbook](#exercise-3-user-management-playbook)
- [Exercise 4: Multi-Directory Structure](#exercise-4-multi-directory-structure)
- [Exercise 5: Configuration File Deployment](#exercise-5-configuration-file-deployment)

---

## Exercise 1: Writing Your First Playbook

### Solution: `day2-playbook.yml`

```yaml
---
- name: Day 2 First Playbook
  hosts: all
  become: true
  tasks:
    - name: Create /tmp/day2-playbook directory
      file:
        path: /tmp/day2-playbook
        state: directory
        mode: '0755'

    - name: Create welcome.txt file with content
      copy:
        content: "Hello from Ansible Day 2"
        dest: /tmp/day2-playbook/welcome.txt
        mode: '0644'
```

### Run the playbook:

```bash
ansible-playbook day2-playbook.yml
```

### Expected Output (First Run):

```
PLAY [Day 2 First Playbook] *******************************************

TASK [Gathering Facts] ************************************************
ok: [node1]
ok: [node2]

TASK [Create /tmp/day2-playbook directory] ****************************
changed: [node1]
changed: [node2]

TASK [Create welcome.txt file with content] ***************************
changed: [node1]
changed: [node2]

PLAY RECAP ************************************************************
node1    : ok=3    changed=2    unreachable=0    failed=0    skipped=0
node2    : ok=3    changed=2    unreachable=0    failed=0    skipped=0
```

### Run the playbook again:

```bash
ansible-playbook day2-playbook.yml
```

### Expected Output (Second Run):

```
PLAY [Day 2 First Playbook] *******************************************

TASK [Gathering Facts] ************************************************
ok: [node1]
ok: [node2]

TASK [Create /tmp/day2-playbook directory] ****************************
ok: [node1]
ok: [node2]

TASK [Create welcome.txt file with content] ***************************
ok: [node1]
ok: [node2]

PLAY RECAP ************************************************************
node1    : ok=3    changed=0    unreachable=0    failed=0    skipped=0
node2    : ok=3    changed=0    unreachable=0    failed=0    skipped=0
```

### Observations:
- **First Run**: Tasks show `changed` status - Ansible created the directory and file
- **Second Run**: Tasks show `ok` status - Resources already in desired state
- **changed=0** on second run demonstrates idempotence

### Verify the results:

```bash
ansible all -m command -a "ls -la /tmp/day2-playbook/"
ansible all -m command -a "cat /tmp/day2-playbook/welcome.txt"
```

---

## Exercise 2: HTTPD Installation

### Solution: `httpd-setup.yml`

```yaml
---
- name: Install and Configure HTTPD
  hosts: all
  become: true
  tasks:
    - name: Install HTTPD package
      yum:
        name: httpd
        state: present

    - name: Start and enable HTTPD service
      service:
        name: httpd
        state: started
        enabled: true
```

### Run the playbook:

```bash
ansible-playbook httpd-setup.yml
```

### Expected Output (First Run):

```
PLAY [Install and Configure HTTPD] ************************************

TASK [Gathering Facts] ************************************************
ok: [node1]
ok: [node2]

TASK [Install HTTPD package] ******************************************
changed: [node1]
changed: [node2]

TASK [Start and enable HTTPD service] *********************************
changed: [node1]
changed: [node2]

PLAY RECAP ************************************************************
node1    : ok=3    changed=2    unreachable=0    failed=0    skipped=0
node2    : ok=3    changed=2    unreachable=0    failed=0    skipped=0
```

### Verify HTTPD is running:

```bash
# Check service status
ansible all -m command -a "systemctl status httpd" --become

# Check if HTTPD is listening on port 80
ansible all -m shell -a "ss -tuln | grep :80"

# Test HTTP response
ansible all -m uri -a "url=http://localhost return_content=yes"
```

### Run the playbook again to test idempotence:

```bash
ansible-playbook httpd-setup.yml
```

### Expected Output (Second Run):

```
PLAY [Install and Configure HTTPD] ************************************

TASK [Gathering Facts] ************************************************
ok: [node1]
ok: [node2]

TASK [Install HTTPD package] ******************************************
ok: [node1]
ok: [node2]

TASK [Start and enable HTTPD service] *********************************
ok: [node1]
ok: [node2]

PLAY RECAP ************************************************************
node1    : ok=3    changed=0    unreachable=0    failed=0    skipped=0
node2    : ok=3    changed=0    unreachable=0    failed=0    skipped=0
```

### Observations:
- HTTPD package is installed
- Service is running and enabled on boot
- Second run shows `changed=0` (idempotent)

---

## Exercise 3: User Management Playbook

### Solution: `user-management.yml`

```yaml
---
- name: User and Group Management
  hosts: all
  become: true
  tasks:
    - name: Create developers group
      group:
        name: developers
        state: present

    - name: Create devops user
      user:
        name: devops
        state: present
        home: /home/devops
        shell: /bin/bash
        groups: developers
        create_home: yes
```

### Run the playbook:

```bash
ansible-playbook user-management.yml
```

### Expected Output (First Run):

```
PLAY [User and Group Management] **************************************

TASK [Gathering Facts] ************************************************
ok: [node1]
ok: [node2]

TASK [Create developers group] ****************************************
changed: [node1]
changed: [node2]

TASK [Create devops user] *********************************************
changed: [node1]
changed: [node2]

PLAY RECAP ************************************************************
node1    : ok=3    changed=2    unreachable=0    failed=0    skipped=0
node2    : ok=3    changed=2    unreachable=0    failed=0    skipped=0
```

### Verify user and group creation:

```bash
# Check if user exists
ansible all -m command -a "id devops" --become

# Check user details
ansible all -m command -a "getent passwd devops" --become

# Check group membership
ansible all -m command -a "groups devops" --become

# Check home directory
ansible all -m command -a "ls -ld /home/devops" --become
```

### Expected verification output:

```
node1 | CHANGED | rc=0 >>
uid=1001(devops) gid=1001(devops) groups=1001(devops),1002(developers)
```

### Run the playbook again to test idempotence:

```bash
ansible-playbook user-management.yml
```

### Expected Output (Second Run):

```
PLAY [User and Group Management] **************************************

TASK [Gathering Facts] ************************************************
ok: [node1]
ok: [node2]

TASK [Create developers group] ****************************************
ok: [node1]
ok: [node2]

TASK [Create devops user] *********************************************
ok: [node1]
ok: [node2]

PLAY RECAP ************************************************************
node1    : ok=3    changed=0    unreachable=0    failed=0    skipped=0
node2    : ok=3    changed=0    unreachable=0    failed=0    skipped=0
```

### Observations:
- Group created first (users need group to exist)
- User created with specified attributes
- Second run shows idempotence

---

## Exercise 4: Multi-Directory Structure

### Solution: `directory-structure.yml`

```yaml
---
- name: Create Multi-Directory Structure
  hosts: all
  become: true
  tasks:
    - name: Create /opt/app directory
      file:
        path: /opt/app
        state: directory
        mode: '0755'

    - name: Create /opt/app/config directory
      file:
        path: /opt/app/config
        state: directory
        mode: '0755'

    - name: Create /opt/app/logs directory
      file:
        path: /opt/app/logs
        state: directory
        mode: '0755'

    - name: Create application.log file
      file:
        path: /opt/app/logs/application.log
        state: touch
        mode: '0644'
```

### Alternative Solution (More Efficient):

```yaml
---
- name: Create Multi-Directory Structure
  hosts: all
  become: true
  tasks:
    - name: Create /opt/app/config directory (creates parent dirs)
      file:
        path: /opt/app/config
        state: directory
        mode: '0755'

    - name: Create /opt/app/logs directory
      file:
        path: /opt/app/logs
        state: directory
        mode: '0755'

    - name: Create application.log file
      file:
        path: /opt/app/logs/application.log
        state: touch
        mode: '0644'
```

**Note:** The `file` module with `state=directory` automatically creates parent directories if they don't exist.

### Run the playbook:

```bash
ansible-playbook directory-structure.yml
```

### Expected Output:

```
PLAY [Create Multi-Directory Structure] *******************************

TASK [Gathering Facts] ************************************************
ok: [node1]
ok: [node2]

TASK [Create /opt/app directory] **************************************
changed: [node1]
changed: [node2]

TASK [Create /opt/app/config directory] *******************************
changed: [node1]
changed: [node2]

TASK [Create /opt/app/logs directory] *********************************
changed: [node1]
changed: [node2]

TASK [Create application.log file] ************************************
changed: [node1]
changed: [node2]

PLAY RECAP ************************************************************
node1    : ok=5    changed=4    unreachable=0    failed=0    skipped=0
node2    : ok=5    changed=4    unreachable=0    failed=0    skipped=0
```

### Verify the directory structure:

```bash
# Check directory structure
ansible all -m command -a "tree /opt/app" --become

# Alternative if tree is not installed
ansible all -m command -a "ls -lR /opt/app" --become

# Check specific permissions
ansible all -m command -a "ls -ld /opt/app /opt/app/config /opt/app/logs" --become
ansible all -m command -a "ls -l /opt/app/logs/application.log" --become
```

### Expected verification output:

```
node1 | CHANGED | rc=0 >>
/opt/app
├── config
└── logs
    └── application.log
```

---

## Exercise 5: Configuration File Deployment

### Solution: `deploy-config.yml`

```yaml
---
- name: Deploy Configuration File
  hosts: all
  become: true
  tasks:
    - name: Create /etc/myapp directory
      file:
        path: /etc/myapp
        state: directory
        mode: '0755'

    - name: Create config.conf file
      copy:
        content: |
          APP_NAME=MyApplication
          APP_PORT=8080
          APP_ENV=production
          LOG_LEVEL=INFO
        dest: /etc/myapp/config.conf
        mode: '0644'
```

**Note:** The `|` (pipe) character in YAML preserves line breaks, making the configuration file more readable.

### Alternative Solution (Single Line):

```yaml
---
- name: Deploy Configuration File
  hosts: all
  become: true
  tasks:
    - name: Create /etc/myapp directory
      file:
        path: /etc/myapp
        state: directory
        mode: '0755'

    - name: Create config.conf file
      copy:
        content: "APP_NAME=MyApplication\nAPP_PORT=8080\nAPP_ENV=production\nLOG_LEVEL=INFO\n"
        dest: /etc/myapp/config.conf
        mode: '0644'
```

### Run the playbook:

```bash
ansible-playbook deploy-config.yml
```

### Expected Output:

```
PLAY [Deploy Configuration File] **************************************

TASK [Gathering Facts] ************************************************
ok: [node1]
ok: [node2]

TASK [Create /etc/myapp directory] ************************************
changed: [node1]
changed: [node2]

TASK [Create config.conf file] ****************************************
changed: [node1]
changed: [node2]

PLAY RECAP ************************************************************
node1    : ok=3    changed=2    unreachable=0    failed=0    skipped=0
node2    : ok=3    changed=2    unreachable=0    failed=0    skipped=0
```

### Verify the configuration file:

```bash
# Check if directory exists
ansible all -m command -a "ls -ld /etc/myapp" --become

# Check file permissions
ansible all -m command -a "ls -l /etc/myapp/config.conf" --become

# View file contents
ansible all -m command -a "cat /etc/myapp/config.conf" --become
```

### Expected verification output:

```
node1 | CHANGED | rc=0 >>
APP_NAME=MyApplication
APP_PORT=8080
APP_ENV=production
LOG_LEVEL=INFO
```

### Test configuration file parsing:

```bash
# Test reading specific config values
ansible all -m shell -a "grep APP_NAME /etc/myapp/config.conf" --become
ansible all -m shell -a "grep APP_PORT /etc/myapp/config.conf" --become
```