# RHCE - Cron Module & Ansible Roles

## 📋 Table of Contents
- [Ansible Cron Module](#ansible-cron-module)
- [Online Roles (Internet Available)](#online-roles-internet-available)
- [Offline Roles (No Internet)](#offline-roles-no-internet)

---

## ⏰ Ansible Cron Module

### Basic Syntax
```yaml
- name: Manage cron job
  cron:
    name: "job description"
    minute: "0"
    hour: "2"
    day: "*"
    month: "*"
    weekday: "*"
    job: "/path/to/command"
    user: "username"
    state: present
```

### Complete Examples

#### 1. Create Daily Backup Cron Job
```yaml
---
- name: Configure cron jobs
  hosts: all
  become: yes
  
  tasks:
    - name: Schedule daily backup at 2:30 AM
      cron:
        name: "Daily backup"
        minute: "30"
        hour: "2"
        job: "/usr/local/bin/backup.sh"
        user: root
        state: present
```

#### 2. Multiple Cron Jobs
```yaml
---
- name: Setup multiple cron jobs
  hosts: webservers
  become: yes
  
  tasks:
    - name: Clear temp files every hour
      cron:
        name: "Clear temp files"
        minute: "0"
        job: "rm -rf /tmp/*"
        user: root
        state: present

    - name: Update system every Sunday at 3 AM
      cron:
        name: "Weekly system update"
        minute: "0"
        hour: "3"
        weekday: "0"
        job: "yum update -y"
        user: root
        state: present

    - name: Database backup every 6 hours
      cron:
        name: "Database backup"
        minute: "0"
        hour: "*/6"
        job: "/usr/local/bin/db-backup.sh"
        user: dbadmin
        state: present

    - name: Log rotation on 1st of every month
      cron:
        name: "Monthly log rotation"
        minute: "0"
        hour: "0"
        day: "1"
        job: "/usr/sbin/logrotate /etc/logrotate.conf"
        user: root
        state: present
```

#### 3. Special Time Values
```yaml
---
- name: Cron with special time values
  hosts: all
  become: yes
  
  tasks:
    - name: Run script at reboot
      cron:
        name: "Run at reboot"
        special_time: reboot
        job: "/usr/local/bin/startup.sh"
        user: root

    - name: Run script hourly
      cron:
        name: "Hourly task"
        special_time: hourly
        job: "/usr/local/bin/hourly-task.sh"
        user: root

    - name: Run script daily
      cron:
        name: "Daily task"
        special_time: daily
        job: "/usr/local/bin/daily-task.sh"
        user: root

    - name: Run script weekly
      cron:
        name: "Weekly task"
        special_time: weekly
        job: "/usr/local/bin/weekly-task.sh"
        user: root

    - name: Run script monthly
      cron:
        name: "Monthly task"
        special_time: monthly
        job: "/usr/local/bin/monthly-task.sh"
        user: root
```

#### 4. Remove Cron Jobs
```yaml
---
- name: Remove cron jobs
  hosts: all
  become: yes
  
  tasks:
    - name: Remove old backup job
      cron:
        name: "Old backup job"
        state: absent
```

---

## 🌐 Offline Roles

### Role 1: Apache Web Server

#### Create role structure
```bash
ansible-galaxy init apache
```

#### roles/apache/tasks/main.yml
```yaml
---
- name: Install Apache
  yum:
    name: httpd
    state: present

- name: Start and enable Apache
  service:
    name: httpd
    state: started
    enabled: yes

- name: Configure firewall
  firewalld:
    service: "{{ item }}"
    permanent: yes
    state: enabled
    immediate: yes
  loop:
    - http
    - https

- name: Deploy website
  template:
    src: index.html.j2
    dest: /var/www/html/index.html
    setype: httpd_sys_content_t
  notify: restart apache
```

#### roles/apache/handlers/main.yml
```yaml
---
- name: restart apache
  service:
    name: httpd
    state: restarted
```

#### roles/apache/templates/index.html.j2
```jinja
{% for host in groups.all %}
    Hello from {{ hostvars[host].ansible_hostname }}, {{ hostvars[host].ansible_default_ipv4.address }}
{% endfor %}
```

---

## 🌐 Online Roles

### Role 1: Install geerlingguy.nginx Role using requirements.yaml

**requirements.yaml**

```yaml
- src: geerlingguy.mysql
  name: mysql
```

```bash
    ansible-galaxy install -r requirements.yaml -p ./myroles
```

### Playbook Using Roles

#### site.yml
```yaml
---
- name: Configure web servers
  hosts: webservers
  become: yes
  roles:
    - apache
    - mysql
```

---

**Dont Forget To Add The Roles Path To the ansible.cfg file Or Else The Error Wont Be Correctly Imported**

```ini
[defaults]
remote_user=ansible
inventory=/home/ansible/inventory
roles_path=/home/ansible/myroles:/usr/share/ansible/roles:/etc/ansible/roles
[privilege_escalation]
become=true
```