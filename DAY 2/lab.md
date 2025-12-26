# Day 2 Lab: Ad-hoc Commands (Exercises)

## Objective
Practice using Ansible ad-hoc commands. Focus on understanding the `state` parameter and idempotence.

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
2. Use an ad-hoc command to Create an empty file `/tmp/day2-lab/test.txt`.
3. Remove the file and directory using ad-hoc commands.

---

### Exercise 3: Package Installation

**Tasks:**
1. Check if the package `vim` is installed on all managed nodes.
2. If not installed, install it using an ad-hoc command.
3. Remove the package after testing, also using an ad-hoc command.
