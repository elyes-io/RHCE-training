# Day 2 Lab: Solutions and Answers

## Table of Contents
- [Exercise 1: Test Connectivity](#exercise-1-test-connectivity)
- [Exercise 2: File and Directory Management](#exercise-2-file-and-directory-management)
- [Exercise 3: Package Installation](#exercise-3-package-installation)
- [Exercise 4: Writing Your First Playbook](#exercise-4-writing-your-first-playbook)
- [Exercise 5: HTTPD Installation Playbook](#exercise-5-httpd-installation-playbook)

---

## Exercise 1: Test Connectivity

### Task 1: Check connectivity to all managed nodes

**Command:**
```bash
ansible all -m ping
```

**Expected Output:**
```
node1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
node2 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

**Explanation:**
- The `ping` module tests connectivity and authentication to managed nodes
- `SUCCESS` means the node is reachable
- `"ping": "pong"` confirms successful connection

---

### Task 2: Check the managed node hostname

**Command:**
```bash
ansible all -m command -a "hostname"
```

**Expected Output:**
```
node1 | CHANGED | rc=0 >>
node1.example.com

node2 | CHANGED | rc=0 >>
node2.example.com
```

**Explanation:**
- The `command` module runs a command on remote nodes
- `-a "hostname"` passes the hostname command as an argument
- Returns the hostname of each managed node

---

## Exercise 2: File and Directory Management

### Task 1: Create a directory `/tmp/day2-lab`

**Command:**
```bash
ansible all -m file -a "path=/tmp/day2-lab state=directory mode=0755"
```

**Expected Output:**
```
node1 | CHANGED => {
    "changed": true,
    "path": "/tmp/day2-lab",
    "state": "directory"
}
```

**Explanation:**
- `file` module manages files and directories
- `state=directory` ensures a directory exists
- `mode=0755` sets permissions (owner: rwx, group: rx, others: rx)

---

### Task 2: Create an empty file `/tmp/day2-lab/test.txt`

**Command:**
```bash
ansible all -m file -a "path=/tmp/day2-lab/test.txt state=touch"
```

**Expected Output:**
```
node1 | CHANGED => {
    "changed": true,
    "dest": "/tmp/day2-lab/test.txt",
    "state": "file"
}
```

**Explanation:**
- `state=touch` creates an empty file if it doesn't exist

**Verify the file was created:**
```bash
ansible all -m command -a "ls -la /tmp/day2-lab/test.txt"
```

---

### Task 3: Remove the file and directory

**Step 1: Remove the file**
```bash
ansible all -m file -a "path=/tmp/day2-lab/test.txt state=absent"
```

**Step 2: Remove the directory**
```bash
ansible all -m file -a "path=/tmp/day2-lab state=absent"
```

**Expected Output:**
```
node1 | CHANGED => {
    "changed": true,
    "path": "/tmp/day2-lab",
    "state": "absent"
}
```

**Explanation:**
- `state=absent` ensures the file or directory does not exist
- Removes it if present, does nothing if already absent
- Can remove directories even if they contain files

---

## Exercise 3: Package Installation

### Task 1: Check if vim is installed

**Command:**
```bash
ansible all -m command -a "which vim"
```

**Alternative command:**
```bash
ansible all -m shell -a "rpm -qa | grep vim"
```

**Expected Output (if installed):**
```
node1 | CHANGED | rc=0 >>
/usr/bin/vim
```

**Expected Output (if not installed):**
```
node1 | FAILED | rc=1 >>
(no output - command returns error)
```

---

### Task 2: Install vim using ad-hoc command

**Command:**
```bash
ansible all -m yum -a "name=vim state=present" --become
```

**Expected Output:**
```
node1 | CHANGED => {
    "changed": true,
    "msg": "Package vim installed successfully"
}
```

**Explanation:**
- `yum` module manages packages on RHEL/CentOS/Rocky Linux
- `state=present` ensures the package is installed
- `--become` elevates privileges (runs as sudo/root)

**Verify installation:**
```bash
ansible all -m command -a "vim --version"
```

---

### Task 3: Remove vim package

**Command:**
```bash
ansible all -m yum -a "name=vim state=absent" --become
```

**Expected Output:**
```
node1 | CHANGED => {
    "changed": true,
    "msg": "Package vim removed successfully"
}
```

**Explanation:**
- `state=absent` ensures the package is not installed
- Removes it if present, does nothing if already removed

---

## Exercise 4: Writing Your First Playbook

### Create the playbook file: `day2-playbook.yml`

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

---

### Run the playbook

**Command:**
```bash
ansible-playbook day2-playbook.yml
```

**Expected Output (First Run):**
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

---

### Run the playbook again (Test Idempotence)

**Command:**
```bash
ansible-playbook day2-playbook.yml
```

**Expected Output (Second Run):**
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

---

### Understanding the Output

**First Run:**
- Tasks show `changed` status (yellow/orange in terminal)
- Ansible made modifications to reach desired state
- `changed=2` in PLAY RECAP shows 2 tasks made changes

**Second Run:**
- Tasks show `ok` status (green in terminal)
- No changes needed - system already in desired state
- `changed=0` in PLAY RECAP shows no changes made

**This demonstrates idempotence:**
- Running the same playbook multiple times produces the same result
- Safe to run repeatedly without side effects
- Only makes changes when necessary

---

## Exercise 5: HTTPD Installation Playbook

### Create the playbook file: `httpd-playbook.yml`

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

---

### Run the playbook

**Command:**
```bash
ansible-playbook httpd-playbook.yml
```

**Expected Output:**
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

---

### Verify HTTPD is running

**Check service status:**
```bash
ansible all -m command -a "systemctl status httpd" --become
```

**Check if HTTPD is listening on port 80:**
```bash
ansible all -m command -a "ss -tuln | grep :80"
```

**Test HTTP access:**
```bash
ansible all -m uri -a "url=http://localhost return_content=yes"
```

**Alternative verification:**
```bash
ansible all -m shell -a "curl http://localhost"
```

---

### Understanding the Playbook

**Task 1: Install HTTPD package**
- `yum` module installs packages on RHEL-based systems
- `name: httpd` specifies the package name
- `state: present` ensures the package is installed

**Task 2: Start and enable HTTPD service**
- `service` module manages system services
- `state: started` ensures the service is running
- `enabled: true` ensures service starts automatically on boot

---

### Test Idempotence

**Run the playbook again:**
```bash
ansible-playbook httpd-playbook.yml
```

**Expected Output (Second Run):**
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

**Observations:**
- All tasks show `ok` status
- `changed=0` means no modifications were made
- HTTPD is already installed and running
- This confirms idempotent behavior

---
