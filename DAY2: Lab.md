# Day 2 Lab: Ad-hoc Commands & Basic Playbooks (Exercises)

## Objective
Practice using Ansible ad-hoc commands and basic YAML playbooks. Focus on understanding the `state` parameter and idempotence.

---

## Exercises

### Exercise 1: Test Connectivity

**Tasks:**
1. Use an ad-hoc command to check connectivity to all managed nodes.
2. Use an ad-hoc command to check the managed node hostname.
---

### Exercise 2: File and Directory Management

**Tasks:**
1. Use an ad-hoc command to create a directory `/tmp/day2-lab` on all managed nodes.
2. Create an empty file `/tmp/day2-lab/test.txt`.
3. Remove the file and directory using ad-hoc commands.

---

### Exercise 3: Package Installation

**Tasks:**
1. Check if the package `vim` is installed on all managed nodes.
2. If not installed, install it using an ad-hoc command.
3. Remove the package after testing, also using an ad-hoc command.

---

### Exercise 4: Writing Your First Playbook

**Tasks:**
Write a YAML playbook to perform the following tasks on all managed nodes:
- Create a directory `/tmp/day2-playbook`.
- Create a file `/tmp/day2-playbook/welcome.txt` with content "Hello from Ansible Day 2".

**Instructions:**
1. Create a playbook file named `day2-playbook.yml`
2. Run the playbook and observe which tasks are `changed` or `ok`.
3. Run the playbook a second time and note the differences.

---

### Exercise 5:

**Tasks:**
Write a YAML playbook to perform the following tasks on all managed nodes:
- Install the HTTPD package.
- Start and Enable the HTTPD package.
