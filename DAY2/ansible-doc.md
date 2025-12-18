# Ansible Documentation Tool (ansible-doc)

## Overview

`ansible-doc` is a powerful command-line tool that provides documentation for Ansible modules directly from your terminal. It's an essential tool for quickly referencing module parameters, examples, and usage without leaving your command line or opening a web browser.

---

## Why Use ansible-doc?

- **Offline Access**: Documentation available without internet connection
- **Fast Reference**: Quickly lookup module syntax and parameters
- **Always Updated**: Documentation matches your installed Ansible version
- **Examples Included**: Real-world examples for each module
- **Terminal-Based**: No need to switch to a browser

---

## Basic Usage

### View Documentation for a Module

**Syntax:**
```bash
ansible-doc <module-name>
```

**Examples:**
```bash
# View documentation for file module
ansible-doc file

# View documentation for yum module
ansible-doc yum

# View documentation for service module
ansible-doc service

# View documentation for copy module
ansible-doc copy
```