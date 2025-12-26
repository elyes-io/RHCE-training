# Day 2 Lab: Solutions and Answers

## Table of Contents
- [Exercise 1: Test Connectivity](#exercise-1-test-connectivity)
- [Exercise 2: File and Directory Management](#exercise-2-file-and-directory-management)
- [Exercise 3: Package Installation](#exercise-3-package-installation)

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