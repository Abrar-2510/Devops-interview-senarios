# Ansible Interview Questions – Explained with Examples

## 1. What is Configuration Management?

Configuration management is the process of maintaining systems such as servers, software, and infrastructure in a consistent and desired state automatically.

Example:
Instead of manually installing packages on 100 servers, configuration management tools install and configure them automatically.

Tools:

* Ansible
* Puppet
* Chef
* SaltStack

When to use:

* Managing many servers
* Automating server setup
* Maintaining consistent environments

When not necessary:

* Very small environments with 1–2 servers

---

## 2. Why Ansible over other configuration management tools?

Reasons:

* Agentless (no agent installation required)
* Uses SSH
* Simple YAML syntax
* Easy to learn

Example:
Using Ansible you can install nginx on multiple servers with one playbook.

When to use:

* Quick automation
* Cloud infrastructure
* DevOps pipelines

When not ideal:

* Very large environments requiring heavy event-driven automation

---

## 3. Explain an Ansible Playbook you wrote

Example Playbook: Install and start nginx

Steps:

1. Update packages
2. Install nginx
3. Start service

Example:

* name: Install nginx
  hosts: webservers
  become: yes

  tasks:

  * name: install nginx
    apt:
    name: nginx
    state: present

  * name: start nginx
    service:
    name: nginx
    state: started

Why effective:

* Automated installation
* Ensures service always running

---

## 4. How Ansible helped your organization?

Example:

* Automated server provisioning
* Reduced manual errors
* Faster deployments

Real scenario:
Deploying application on 20 servers using one playbook.

---

## 5. Managing future servers you don't know yet

Solution: Dynamic Inventory

Example:
AWS dynamic inventory automatically discovers new EC2 instances.

Why useful:
Servers created later are automatically managed.

---

## 6. What is Ansible Tower?

Ansible Tower is a web-based UI for managing Ansible automation.

Features:

* RBAC
* Job scheduling
* Logging
* Inventory management

When to use:
Large teams and enterprise environments.

---

## 7. Managing RBAC in Ansible Tower

RBAC controls who can:

* run playbooks
* edit inventories
* manage credentials

Example roles:

* Admin
* Operator
* Developer

---

## 8. What is Ansible Galaxy?

Galaxy is a repository for sharing Ansible roles.

Example:
Installing nginx role

ansible-galaxy install geerlingguy.nginx

When to use:
Reuse community roles instead of writing everything.

---

## 9. Structure of Playbook using Roles

Roles help organize large playbooks.

Example structure:

roles/
web/
tasks/
handlers/
templates/
vars/

Benefits:
Reusable and organized automation.

---

## 10. What are Handlers?

Handlers run only when notified by a task.

Example:
Restart nginx only if config file changes.

---

## 11. Running tasks only on Windows

Use condition:

when: ansible_os_family == "Windows"

Example:
Install IIS only on Windows machines.

summery

We usually separate Windows and Linux hosts using inventory groups. However, we can also use conditions like when: ansible_os_family == "Windows" when a playbook targets multiple operating systems. Tags are mainly used to control which tasks run during execution, not for OS detection.

---

## 12. Parallel execution in Ansible

Ansible runs tasks in parallel by default using forks.

Example:
Running a playbook on 20 servers simultaneously.

---

## 13. Protocol used for Windows

Ansible uses:

WinRM (Windows Remote Management)

---

## 14. Variable precedence in Ansible

Order from lowest to highest priority:

* role defaults
* inventory vars
* play vars
* extra vars

Extra vars override everything.

Role defaults provide fallback values for roles and are easy to override.
Inventory variables define environment-specific settings for hosts or groups.
Play variables are defined inside the playbook and control the behavior of that specific play execution.
Extra variables have the highest priority because they are passed at runtime by the user, allowing them to override any other variable without modifying the code.

---

## 15. Handling secrets in Ansible

Use:

Ansible Vault

Example:
Encrypt passwords and API keys.

Command:

ansible-vault encrypt secrets.yml

---

## 16. Can Ansible be used for IaC?

Yes.

Example:
Creating AWS EC2 instances using Ansible modules.

Used together with:

* Terraform
* Cloud APIs

---

## 17. Playbook to install httpd on EC2

Example:

* name: install httpd
  hosts: web
  become: yes

  tasks:

  * name: install apache
    yum:
    name: httpd
    state: present

  * name: start service
    service:
    name: httpd
    state: started

---

## 18. What can Ansible improve?

Possible improvements:

* Better error messages
* Faster execution for large infrastructures
* Improved Windows support


---
## 19.Question Can you talk about an Ansible playbook that you wrote and how it helped your company?

Answer

Answer

Yes. 
I wrote an Ansible playbook to automate the installation of a Kubernetes cluster using kubeadm. The playbook prepares the nodes by installing containerd, configuring system settings, and installing Kubernetes components. I used roles to separate master and worker configurations, templates for dynamic configuration files, handlers to restart services when configuration changes, and Galaxy roles to reuse existing automation. This significantly reduced the time required to set up clusters and ensured consistent infrastructure configuration.

Using this playbook, we were able to deploy a fully configured Kubernetes cluster in a few minutes instead of manually configuring each node. This improved consistency across environments and significantly reduced setup time for development and testing clusters.

# Dynamic Inventory (Markdown Section)

## What is Dynamic Inventory?

Dynamic inventory means the host list is **generated automatically** (e.g., from AWS/GCP/Azure) instead of a static `inventory.ini`.

### Static vs Dynamic

* **Static:** you manually add IPs/hosts
* **Dynamic:** Ansible discovers instances based on filters/tags

### Why it’s useful

* Auto Scaling groups
* Instances created/terminated frequently
* No manual inventory updates

## AWS Dynamic Inventory Example

### Install AWS collection

* `ansible-galaxy collection install amazon.aws`

### Inventory config file: `aws_ec2.yml`

```yaml
plugin: amazon.aws.aws_ec2
regions:
  - us-east-1

filters:
  instance-state-name: running

keyed_groups:
  - key: tags.Role
    prefix: tag_Role
```

### Test inventory

* `ansible-inventory -i aws_ec2.yml --graph`

### Use in a playbook

```yaml
- name: Configure web servers
  hosts: tag_Role_web
  become: yes

  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present
```

## When to use Dynamic Inventory

* Cloud environments
* Auto Scaling
* Ephemeral workloads

## When static is enough

* Fixed on-prem servers
* Small infra with rare changes

---

# Ansible Role Structure – Interview Explanation

This guide explains the main directories inside an **Ansible role** and why we use each one. It is written in a way that can be used directly in interviews.

---

# Example Role Structure

roles/
web/
tasks/
handlers/
templates/
vars/

---

# 1. roles/

## What is roles/?

The **roles/** directory contains all the roles used in an Ansible project.

Roles help organize automation into **modular and reusable components**.

Example:

roles/
web/
database/
monitoring/

Each role manages a specific part of the infrastructure.

## Why we use roles

* Organize large playbooks
* Reuse automation code
* Make projects easier to maintain

## Interview Answer

Roles allow us to break down complex automation into reusable and organized components such as web, database, or monitoring roles.

---

# 2. web/

The **web/** directory is a specific role responsible for configuring web servers.

Example responsibilities:

* Install nginx
* Configure nginx
* Start the nginx service

Example:

roles/
web/

This role manages everything related to web server configuration.

---

# 3. tasks/

## What is tasks/?

The **tasks/** directory contains the main automation steps executed by Ansible.

File usually used:

tasks/main.yml

Example:

* name: Install nginx
  apt:
  name: nginx
  state: present

* name: Start nginx
  service:
  name: nginx
  state: started

## Why we use tasks

Tasks define the actual operations performed on the servers.

## Interview Answer

The tasks directory contains the main automation steps executed by the role.

---

# 4. handlers/

## What are handlers?

Handlers are tasks that run **only when notified by another task**.

They are commonly used to restart services after configuration changes.

Example file:

handlers/main.yml

Example:

* name: restart nginx
  service:
  name: nginx
  state: restarted

Example usage in tasks:

notify: restart nginx

## Why we use handlers

* Avoid unnecessary service restarts
* Improve system stability

## Interview Answer

Handlers are triggered tasks that usually restart services only when configuration changes occur.

---

# 5. templates/

## What is templates/?

The **templates/** directory stores configuration templates written using **Jinja2**.

These templates allow dynamic configuration using variables.

Example file:

templates/nginx.conf.j2

Example content:

server {
listen {{ nginx_port }};
}

Ansible replaces the variable with the actual value during execution.

## Why we use templates

* Dynamic configuration files
* Environment-specific settings

## Interview Answer

Templates allow us to create dynamic configuration files using Jinja2 variables.

---

# 6. vars/

## What is vars/?

The **vars/** directory stores variables used by the role.

Example file:

vars/main.yml

Example:

nginx_port: 80
nginx_user: www-data

These variables can be used in tasks or templates.

## Why we use vars

* Store role-specific configuration values
* Simplify configuration management

## Interview Answer

The vars directory stores variables used by the role such as ports, usernames, and configuration values.

---

# Example Playbook Using the Role

* hosts: webservers
  roles:

  * web

This playbook applies the **web role** to all servers in the webservers group.

---

# Short Interview Summary

Roles help organize Ansible automation into reusable components. Tasks define the main operations, handlers restart services when changes occur, templates generate dynamic configuration files, and vars store role-specific variables.
