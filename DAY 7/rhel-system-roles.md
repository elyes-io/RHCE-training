# RHEL System Roles (rhel-system-roles)

## Overview

**rhel-system-roles** is a collection of Ansible roles provided by **Red Hat** to simplify and standardize the configuration and management of **Red Hat Enterprise Linux (RHEL)** systems.

These roles are:
- Supported by Red Hat
- Designed for enterprise and RHCE use cases

---

## 📦 Installation

Install the roles using `yum` (or `dnf`):

```bash
sudo yum install rhel-system-roles
```

The roles are installed by default in:

```bash
/usr/share/ansible/roles/
```

## Roles Path Configuration

You can either copy the roles to your custom roles directory or add the path directly in ansible.cfg.

**Option 1: Copy roles to a custom directory**

```bash
sudo cp -r /usr/share/ansible/roles/* /home/ansible/myroles
```

**Option 2: Add roles path in ansible.cfg (Recommended)**

```ini
[defaults]
remote_user=ansible
inventory=/home/ansible/inventory
roles_path=/home/ansible/myroles:/usr/share/ansible/roles:/etc/ansible/roles

[privilege_escalation]
become=true
```

## 📚 Documentation Location

Each role includes its own documentation and examples.

Main documentation directory:

```bash
ls /usr/share/doc/rhel-system-roles
```

**NTP - timesync roles**

```yaml
---
- name: Configure time synchronization
  hosts: all
  become: yes
  vars:
    timesync_ntp_servers:
      - hostname: time.example.com
        iburst: yes
  roles:
    - rhel-system-roles.timesync
```

**SELINUX role**

```yaml
---
- name: Manage SELinux policy
  hosts: all
  become: yes
  vars:
    selinux_policy: targeted
    selinux_state: enforcing
  roles:
    - rhel-system-roles.selinux
```