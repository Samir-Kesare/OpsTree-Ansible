# Ansible Role for Fluentd (log aggregator)

<img src="https://github.com/CodeOps-Hub/Ansible/assets/156056349/4ea2fd53-751c-4da8-a3ce-86813667bf91" alt="image1" width="400">  <img src="https://github.com/CodeOps-Hub/Ansible/assets/156056349/5a6ee770-1c53-4950-bb18-34e00f926c25" alt="image2" width="500">



***

| **Author** | **Created on** | **Last Updated** | **Document Version** |
| ---------- | -------------- | ---------------- | -------------------- |
| **Vidhi Yadav** | **14th April** | **14th April** | **v1** |

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

This Ansible role automates the deployment process, ensuring a seamless setup of Fluentd on target servers. Upon execution, the role not only installs Fluentd but also configures it to forward logs to Elasticsearch by default. This default behavior provides immediate functionality out-of-the-box, simplifying the log aggregation process.

***

# Flow Diagram

<img width="1276" alt="Screenshot 2024-04-14 at 3 24 20 PM" src="https://github.com/CodeOps-Hub/Ansible/assets/156056349/e6a34d41-535e-4acb-a1f1-360a129ab23d">

***

# Pre-requisites

| **Pre-requisite** | **Description** |
| ----------------- | --------------- |
| **Ansible**       | Ansible must be installed on the control machine from which you plan to run the playbook. |
| **AWS CLI**       | AWS CLI for providing aws credentials to fetch resources. |
| **Python**        | Ensure that Python is installed on your system. Ansible relies on Python for its execution, and dynamic inventory scripts are typically written in Python. |
| **PIP (Python Package Installer)** | Install pip if it's not already installed. pip is a package manager for Python that allows you to install and manage Python packages. |
| **Boto3**   |  If your dynamic inventory script relies on AWS APIs to fetch inventory data, you'll need to install `boto3` using `pip`. |

***

# Directory Structure

<img width="400" alt="image" src="https://github.com/CodeOps-Hub/Ansible/assets/156057205/d5803b13-5868-4ef6-8535-cf7fffa09390">

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

### fluentd.yml file

<details>
<summary> Click here to see fluentd.yml file</summary>
<br>
  
```shell
---
- hosts: fluent
  become: yes
  gather_facts: yes
  roles:
    - fluentd-collector

```
</details>

***

## Fluentd Role

### defaults file (main.yml)

The defaults/main.yml file helps maintain consistency and simplifies role configuration by defining default values for variables. 

<details>
<summary> Click here to see main.yml file</summary>
<br>
  
```shell
---
# defaults file for fluentd-role
# defaults file for ansible-role-fluentd
host: localhost
service_port: 9200
logstash_format: true
flush_interval: 10s
type : elasticsearch

```
</details>

***

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
  author: Vidhi Yadav
  description: This role facilitates the installation and configuration of Fluentd as a robust log aggregator. It seamlessly integrates with Elasticsearch to store and manage log data efficiently. While Elasticsearch is the default destination, this role offers flexibility, allowing users to override or customize the destination as needed.

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

- name: Check if <match **> exists in the file
  ansible.builtin.shell:
    cmd: "grep '@type {{ type }}' /etc/fluent/fluentd.conf"
  register: match_exists
  changed_when: false
  failed_when: false

- name: Add Fluentd configuration to the end of the file
  ansible.builtin.lineinfile:
    path: /etc/fluent/fluentd.conf
    insertafter: EOF
    line: |
      <match **>
        @type {{ type }}
        host {{ host }}
        port {{ service_port }}
        logstash_format {{ logstash_format | lower}}
        flush_interval {{ flush_interval }}
      </match>
  when: match_exists.rc != 0
  notify: Restart Fluentd service

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

***

# Output

### Flunetd Role Execution

<img width="1601" alt="Screenshot 2024-04-14 at 2 43 31 PM" src="https://github.com/CodeOps-Hub/Ansible/assets/156056349/f59c911e-e3e7-45a6-8b3b-b9fc4ed9b6f0">

***

### Target Server 

<img width="1124" alt="Screenshot 2024-04-14 at 3 38 29 PM" src="https://github.com/CodeOps-Hub/Ansible/assets/156056349/7845cd05-4e68-4cde-ba85-1a9755fa68b6">


***

# Conclusion

This Ansible role for Fluentd collector simplifies the process of deploying and configuring Fluentd on target servers. By utilizing the defined variables in the vars file, the role dynamically selects the appropriate Fluentd package URLs based on the target server's Linux distribution and version, ensuring a seamless installation process.

Once installed, Fluentd acts as a robust log aggregator, collecting logs from various sources and forwarding them to Elasticsearch for indexing and analysis. The role's configuration options allow for flexibility, enabling users to customize or override default settings according to their specific requirements.

***

# Contact Information

| **Name** | **Email Address** |
| -------- | ----------------- |
| **Vidhi Yadav** | vidhi.yadhav.snaatak@mygurukulam.co |

***

# References

| **Source** | **Description** |
| ---------- | --------------- |
| [Link](https://docs.fluentd.org/installation/install-by-deb) | Installation |
| [Link](https://docs.fluentd.org/configuration) | Configuration |

