# Ansible Role for Fluentd Setup of Employee API 

<img src="https://github.com/CodeOps-Hub/Ansible/assets/156056349/4ea2fd53-751c-4da8-a3ce-86813667bf91" alt="image1" width="400">  <img src="https://github.com/CodeOps-Hub/Ansible/assets/156056349/5a6ee770-1c53-4950-bb18-34e00f926c25" alt="image2" width="500">

***

| **Author** | **Created on** | **Last Updated** | **Document Version** |
| ---------- | -------------- | ---------------- | -------------------- |
| **Shreya Jaiswal** | **20th April** | **20th April** | **v1** |

***
# Table of contents
* [Introduction](#Introduction)
* [Flow Diagram](#Flow-Diagram)
* [Pre-requisites](#Pre-requisites)
* [Directory Structure](#Directory-Structure)
* [Output](#Output)
* [Conclusion](#Conclusion)
* [Contact Information](#Contact-Information)
* [References](#References)

***

# Introduction

In the context of managing logging for an Employee API, an Ansible role for Fluentd simplifies deployment and configuration. This role automates Fluentd installation, configuration, and deployment on API servers, ensuring consistent, reliable log handling.

***

# Flow Diagram

<img width="564" alt="image" src="https://github.com/CodeOps-Hub/Ansible/assets/156057205/e257b87f-f9f4-4ca7-bb60-67e1f0372401">

***

# Pre-requisites

| **Pre-requisite** | **Description** |
| ----------------- | --------------- |
| **Ansible**       | Ansible must be installed on the control machine from which you plan to run the playbook. |
| **AWS CLI**       | AWS CLI for providing aws credentials to fetch resources. |
| **Python**        | Ensure that Python is installed on your system. Ansible relies on Python for its execution, and dynamic inventory scripts are typically written in Python. |
| **PIP (Python Package Installer)** | Install pip if it's not already installed. pip is a package manager for Python that allows you to install and manage Python packages. |
| **Boto3**   |  If your dynamic inventory script relies on AWS APIs to fetch inventory data, you'll need to install `boto3` using `pip`. |
| **Employee API** | Must have configured Employee API to collect logs. |

***

# Directory Structure

<img width="335" alt="image" src="https://github.com/CodeOps-Hub/Ansible/assets/156057205/69665cd7-0527-4e26-ba71-e61e0d42ed6f">

***

## Dynamic Inventory Setup

### ansible.cfg file

<details>
<summary> Click here to see ansible.cfg file</summary>
<br>
  
```shell
[defaults]

# some basic default values...


# Use AWS EC2 dynamic inventory for managing hosts
inventory      = aws_ec2.yml

# Disable SSH host key checking for convenience.
host_key_checking = False

# Specify the path to the private key file for SSH connections.
private_key_file = /path/to/private_key.pem

#Sets the remote user for SSH connections to 'ubuntu'
remote_user = ubuntu

[inventory]
# enable inventory plugins, default: 'host_list', 'script', 'auto', 'yaml', 'ini', 'toml'
enable_plugins = aws_ec2, host_list, virtualbox, yaml, constructed, script, auto, ini, toml

```
</details>

***

### aws_ec2.yml inventory file

<details>
<summary> Click here to see aws_ec2.yml file</summary>
<br>
  
```shell
---
plugin: aws_ec2
regions:
  - us-east-1

groups: 
  redis: "'redis' in tags.Type"
  postgres: "'postgres' in tags.Type"
  scylla: "'scylla' in tags.Type"
  efk: "'efk' in tags.Type"
  prometheus: "'prometheus' in tags.Type"
  frontend: "'frontend-node-exporter' in tags.Type"
  employee: "'employee' in tags.Type"
  fluent: "'fluent' in tags.Type"

filters:
    instance-state-code: 16

```
</details>

***

### emp-fluentd.yml file

<details>
<summary> Click here to see fluentd.yml file</summary>
<br>
  
```shell
---
- hosts: employee
  become: yes
  gather_facts: yes
  roles:
    - Employee-fluentd-role

```
</details>

***

## Fluentd Role

### handlers file (main.yml)

This Ansible task named "Restart Fluentd service" executes the command systemctl restart fluentd to restart the fluentd service and load its updated configuration. 

<details>
<summary> Click here to see main.yml file</summary>
<br>
  
```shell
---
# handlers file for fluentd-role
- name: Restart Fluentd service
  ansible.builtin.service:
    name: fluentd
    state: restarted

```
</details>

***

### meta file (main.yml)

This is a metadata file (meta/main.yml) for an Ansible role, providing essential information about the role such as author, description and supported platforms.

<details>
<summary> Click here to see main.yml file</summary>
<br>
  
```shell

galaxy_info:
  author: Shreya Jaiswal
  description: Fluentd role for Employee API.

  license: license (GPL-2.0-or-later, MIT, etc)

  min_ansible_version: 2.1

  platforms:
  - name: Ubuntu
    distribution:
    - jammy
    - focal
  - name: Debian
    distribution:
    - bullseye
    - bookworm



```
</details>

***

### tasks file (main.yml)

This playbook automates the setup process, ensuring a smooth installation and configuration of fluentd on the target system.

<details>
<summary> Click here to see main.yml file</summary>
<br>
  
```shell
---
- name: Install Fluentd
  ansible.builtin.shell: "curl -fsSL {{ fluentd_repo_urls[ansible_distribution ~ '_' ~ ansible_distribution_release] }} | sh"

- name: Check status of Fluentd service
  ansible.builtin.service:
    name: fluentd
    state: "started"
  register: fluentd_service_status

- name: Display Fluentd service status
  ansible.builtin.debug:
    msg: "Fluentd service status: {{ fluentd_service_status }}"

- name: Replace Fluentd configuration file
  ansible.builtin.template:
    src: fluentd.conf.j2
    dest: /etc/fluent/fluentd.conf
  notify: Restart Fluentd service

- name: Display Fluentd service status
  ansible.builtin.debug:
    msg: "Fluentd service status: {{ fluentd_service_status }}"
```
</details>

***

### vars file (main.yml)

This vars file contains a set of variables used by the Ansible role to define the URLs for downloading and installing Fluentd packages on different Linux distributions. 

<details>
<summary> Click here to see main.yml file</summary>
<br>
  
```shell

---
# vars file for fluentd-collector
fluentd_repo_urls:
  Ubuntu_jammy: "https://toolbelt.treasuredata.com/sh/install-ubuntu-jammy-fluent-package5-lts.sh"
  Ubuntu_focal: "https://toolbelt.treasuredata.com/sh/install-ubuntu-focal-fluent-package5-lts.sh"
  Debian_bookworm: "https://toolbelt.treasuredata.com/sh/install-debian-bookworm-fluent-package5-lts.sh"
  Debian_bullseye: "https://toolbelt.treasuredata.com/sh/install-debian-bullseye-fluent-package5-lts.sh"

```
</details>


### templates file (fluentd.conf.j2)

This templates file contains the configuration for fluentd as a log aggregator that sends all the logs to elasticsearch.

<details>
<summary> Click here to see fluentd.conf.j2 file</summary>
<br>
  
```shell
<source>
  @type tail
  @id input_log
  <parse>
    @type none
  </parse>
  path /var/log/logfile.log
  tag employee.log
  read_from_head true
 </source>

<match employee.log>
  @type forward
  @id output_system_forward
  <server>
    host 172.31.40.158
    port 24224
  </server>
</match>

```
</details>

***

# Output

### Employee API Status

<img width="1238" alt="Screenshot 2024-04-20 at 10 21 04 PM" src="https://github.com/CodeOps-Hub/Ansible/assets/156057205/821722df-e60a-4566-922e-dc0c8074d9d6">

***

### Flunetd Role Execution

<img width="1238" alt="Screenshot 2024-04-20 at 10 21 04 PM" src="https://github.com/CodeOps-Hub/Ansible/assets/156057205/a808cfbc-8233-4231-8311-227b2e442bb3">

***

### Server Status 

<img width="1124" alt="Screenshot 2024-04-14 at 3 38 29 PM" src="https://github.com/CodeOps-Hub/Ansible/assets/156057205/e1bb88f8-d212-4761-a6bb-8735d1d0b92f">

***

### Logs 

<img width="1124" alt="Screenshot 2024-04-14 at 3 38 29 PM" src="https://github.com/CodeOps-Hub/Ansible/assets/156057205/d74edfe9-4062-4b83-a888-fbf440e5d62d">

***

### 

# Conclusion

Implementing an Ansible role for Fluentd to manage the logging infrastructure for an Employee API offers significant advantages in terms of automation and standardization. By encapsulating the setup and configuration steps into a reusable role, you can easily deploy and manage Fluentd across multiple servers hosting the Employee API service. 

***

# Contact Information

| **Name** | **Email Address** |
| -------- | ----------------- |
| **Shreya Jaiswal** | shreya.jaiswal.snaatak@mygurukulam.co |

***

# References

| **Source** | **Description** |
| ---------- | --------------- |
| [Link](https://docs.fluentd.org/installation/install-by-deb) | Installation |
| [Link](https://docs.fluentd.org/configuration) | Configuration |
