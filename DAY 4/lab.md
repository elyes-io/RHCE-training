# Lab: Ansible Variables, Facts, and Vault

## Lab Overview
In this lab, you will practice:
- Defining and using variables in Ansible playbooks
- Working with external variable files
- Utilizing Ansible facts for system information
- Securing sensitive data using Ansible Vault

---

## Exercise 1: Working with Simple Variables

### Task 1.1: Create a Playbook with Inline Variables

Create a playbook named `install_package.yml` that:
- Targets all hosts
- Defines a variable for package name (e.g., `vim`)
- Installs the package using the appropriate module based on OS family
Example: if the OS family is redhat install vim else skip that task

**Test your playbook:**
- Run the playbook normally

---

## Exercise 2: Using List Variables and Loops

### Task 2.1: Install Multiple Packages

Create a playbook named `install_multiple.yml` that:
- Defines a list variable containing 4 common packages (git, wget, curl, httpd)
- Uses a loop to install all packages

---

## Exercise 3: External Variable Files

### Task 3.1: Create Variable Files

Create the following directory structure:
```
project/
├── vars/
│   ├── users.yml
└── setup_webserver.yml
```

**In `vars/users.yml`**, define variables for:
- Application username
- Application group
- User shell


### Task 3.2: Use Variable Files in Playbook

Create `create.yml` that:
- Loads the users,yml variable file
- Creates the application group
- Creates the application user

---

## Exercise 4: Exploring Ansible Facts

### Task 4.1: View All Facts

Use ad-hoc commands to:
- View all facts for all hosts in your inventory
- Filter facts to show only OS-related information
- Filter facts to show only ipv4 information
- Filter facts to show only memory information

### Task 4.2: Create Playbook Using Facts

Create a playbook named `system_info.yml` that displays:
- Hostname
- OS family
- Distribution and version
- IP address
- Total memory in MB
- CPU core count

The output needs to be on a file that exists on each managed node under /etc/specs.txt.

### Task 4.3: Conditional Execution Based on Facts

Create a playbook named `conditional_install.yml` that:
- Installs Apache web server (httpd) based on OS family
- Shows a message if system has >= 2GB of memory
- Shows a warning if system has < 2GB of memory

---

## Exercise 5: Ansible Vault - Securing Sensitive Data

### Task 5.1: Create Encrypted File

Create an encrypted file named `secrets.yml` containing:
- A database password variable
- An API key variable
- An admin password variable

### Task 5.2: View and Edit Encrypted Files

Try to cat the vault file: 
- Can you explain the output ?
Practice the following Vault operations:
- View the encrypted content
- Edit the encrypted file to add another variable
- View it again to confirm your changes

### Task 5.3: Encrypt Existing File

1. Create a plain file named `database.yml` with database configuration
2. Encrypt the file using Ansible Vault
3. Verify you can no longer read it as plain text

### Task 5.4: Use Vault in Playbook

Create a playbook named `deploy_app.yml` that:
- Loads the secrets.yml file
- Creates an admin user with an encrypted password from vault

Run the playbook using:
- The `--ask-vault-pass` option
- A password file

---