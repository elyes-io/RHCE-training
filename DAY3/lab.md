### Exercise 1: Writing Your First Playbook

**Tasks:**
Write a YAML playbook to perform the following tasks on all managed nodes:
- Create a directory `/tmp/day2-playbook`.
- Create a file `/tmp/day2-playbook/welcome.txt` with content "Hello from Ansible Day 2".

**Instructions:**
1. Create a playbook file named `day2-playbook.yml`
2. Run the playbook and observe which tasks are `changed` or `ok`.
3. Run the playbook a second time and note the differences.

---

### Exercise 2: HTTPD Installation

**Tasks:**
Write a YAML playbook to perform the following tasks on all managed nodes:
- Install the HTTPD package.
- Start and Enable the HTTPD service.

**Instructions:**
1. Create a playbook file named `httpd-setup.yml`
2. Run the playbook and verify HTTPD is running
3. Run the playbook again to test idempotence

---

### Exercise 3: User Management Playbook

**Tasks:**
Write a YAML playbook to perform the following tasks on all managed nodes:
- Create a user named `devops` with home directory `/home/devops`
- Create a group named `developers`
- Add the `devops` user to the `developers` group
- Set the shell for the user to `/bin/bash`

**Instructions:**
1. Create a playbook file named `user-management.yml`
2. Run the playbook and verify the user was created
3. Run the playbook again to test idempotence

---

### Exercise 4: Multi-Directory Structure

**Tasks:**
Write a YAML playbook to perform the following tasks on all managed nodes:
- Create the following directory structure:
  - `/opt/app`
  - `/opt/app/config`
  - `/opt/app/logs`
- Set permissions to `0755` for all directories
- Create an empty file `/opt/app/logs/application.log` with permissions `0644`

**Instructions:**
1. Create a playbook file named `directory-structure.yml`
2. Use the `file` module for all tasks
3. Run the playbook and verify the structure was created

---

### Exercise 5: Configuration File Deployment

**Tasks:**
Write a YAML playbook to perform the following tasks on all managed nodes:
- Create a directory `/etc/myapp`
- Create a configuration file `/etc/myapp/config.conf` with the following content:
```
  APP_NAME=MyApplication
  APP_PORT=8080
  APP_ENV=production
  LOG_LEVEL=INFO
```
- Set file permissions to `0644`

**Instructions:**
1. Create a playbook file named `deploy-config.yml`
2. Use the `copy` module with `content` parameter
3. Run the playbook and verify the configuration file