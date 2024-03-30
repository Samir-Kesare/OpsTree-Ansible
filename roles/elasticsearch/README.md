
# Elasticsearch Ansible Role Documentation

![image](https://github.com/CodeOps-Hub/Ansible/assets/156056746/818655cc-4bab-465b-9faf-0b2004eac4ea)



***

| **Author** | **Created on** | **Last Updated** | **Document Version** |
| ---------- | -------------- | ---------------- | -------------------- |
| **Shikha Tripathi** | **30 March 2024** | **30 March 2024** | **v1** |

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

This Ansible role automates the installation and configuration of Elasticsearch on target hosts. It simplifies the process of deploying and managing Elasticsearch clusters.
***

# Flow Diagram

![image](https://github.com/CodeOps-Hub/Ansible/assets/156056746/7b1918b8-98fd-47e1-a391-e24fee29a54e)

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

![Screenshot from 2024-03-30 15-38-54](https://github.com/CodeOps-Hub/Ansible/assets/156056746/0df7e976-9d76-43c6-8c8d-5ce0067288f2)


## Dynamic Inventory Setup

### ansible.cfg file

<details>
<summary> Click here to see ansible.cfg file</summary>
<br>
  
```shell
[defaults]
roles_path=ansible-role-elasticsearch
retry_files_enabled=no
inventory=aws_ec2.yml
host_key_checking = False
remote_user = ubuntu
private_key_file =/home/ubuntu/elasticSearch.pem
[inventory]
enable_plugins = aws_ec2

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
hostnames:
  - ip-address
filters:
  tag:Name:
    - ElasticSearch-Server
```
</details>

***

### playbook.yml file

<details>
<summary> Click here to see playbook.yml file</summary>
<br>
  
```shell
---
- hosts: aws_ec2
  become: yes
  gather_facts: yes
  roles:
    - ansible-role-elasticsearch

```
</details>

***

## Elasticsearch Role

### defaults file (main.yml)

The defaults/main.yml file helps maintain consistency and simplifies role configuration by defining default values for variables. 

<details>
<summary> Click here to see main.yml file</summary>
<br>
  
```shell
---
# defaults file for ansible-role-elasticsearch
elasticsearch_version: "7.x"
```
</details>

***

### handlers file (main.yml)

This Ansible task named "Reload systemd" executes the command systemctl daemon-reload to reload the systemd configuration. It includes a listen directive with the value systemd_reload, indicating that other tasks can notify this task to trigger a reload of systemd. 

<details>
<summary> Click here to see main.yml file</summary>
<br>
  
```shell
# handlers file for ansible-role-elasticsearch
---
- name: restart elasticsearch
  systemd:
    name: elasticsearch
    state: restarted
    enabled: yes
```
</details>

***

### meta file (main.yml)

This is a metadata file (meta/main.yml) for an Ansible role, providing essential information about the role such as author, description, company, license, and supported platforms.

<details>
<summary> Click here to see main.yml file</summary>
<br>
  
```shell
galaxy_info:
  author: Shikha
  description: Snaatak-P7
  company: Mygurukulam- Powered by Opstree

  # If the issue tracker for your role is not on github, uncomment the
  # next line and provide a value
  # issue_tracker_url: http://example.com/issue/tracker

  # Choose a valid license ID from https://spdx.org - some suggested licenses:
  # - BSD-3-Clause (default)
  # - MIT
  # - GPL-2.0-or-later
  # - GPL-3.0-only
  # - Apache-2.0
  # - CC-BY-4.0
  license: license (GPL-2.0-or-later, MIT, etc)

  min_ansible_version: 2.1

  # If this a Container Enabled role, provide the minimum Ansible Container version.
  # min_ansible_container_version:

  #
  # Provide a list of supported platforms, and for each platform a list of versions.
# If you don't wish to enumerate all versions for a particular platform, use 'all'.
  # To view available platforms and versions (or releases), visit:
  # https://galaxy.ansible.com/api/v1/platforms/
  #
  # platforms:
  # - name: Fedora
  #   versions:
  #   - all
  #   - 25
  # - name: SomePlatform
  #   versions:
  #   - all
  #   - 1.0
  #   - 7
  #   - 99.99

  galaxy_tags: []
# List tags for your role here, one per line. A tag is a keyword that describes
    # and categorizes the role. Users find roles by searching for tags. Be sure to
    # remove the '[]' above, if you add tags to this list.
    #
    # NOTE: A tag is limited to a single word comprised of alphanumeric characters.
    #       Maximum 20 tags per role.

dependencies: []
  # List your role dependencies here, one per line. Be sure to remove the '[]' above,
  # if you add dependencies to this list.
```
</details>

***

### tasks file (main.yml)

This playbook automates the setup process, ensuring a smooth installation and configuration of Elasticsearch on the target system.

<details>
<summary> Click here to see main.yml file</summary>
<br>
  
```shell

# tasks file for ansible-role-elasticsearch
---
- name: Add Elasticsearch GPG Key
  apt_key:
    url: https://artifacts.elastic.co/GPG-KEY-elasticsearch
  notify: restart elasticsearch

- name: Add Elasticsearch APT Repository
  apt_repository:
    repo: "deb https://artifacts.elastic.co/packages/{{ elasticsearch_version }}/apt stable main"
    state: present
  notify: restart elasticsearch

- name: Update apt and install Elasticsearch
  apt:
    name: elasticsearch
    state: latest
  notify: restart elasticsearch

- name: Copy Elasticsearch configuration template
  template:
    src: elasticsearch.j2
    dest: /etc/elasticsearch/elasticsearch.yml
  notify: restart elasticsearch
```
</details>

***

### templates file (elasticsearch.j2)

This configuration file is for setting up and managing the Elasticsearch server, defines the behavior of the Elasticsearch service, including how it should be started, restarted.

<details>
<summary> Click here to see prometheus_config.j2 file</summary>
<br>
  
```shell
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Hello Elasticsearch</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #f4f4f4;
            margin: 0;
            padding: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
        }
        .container {
            text-align: center;
        }
        h1 {
            color: #333;
        }
        p {
            color: #666;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Hello Elasticsearch!</h1>
        <p>Welcome to our Elasticsearch dashboard.</p>
    </div>
</body>
</html>

```
</details>

***

### tests file (test.yml)

This Ansible playbook named "Install Elasticsearch" targets the cloud server machine and utilizes the "Elasticsearch_role" role to install Elasticsearch

<details>
<summary> Click here to see test.yml file</summary>
<br>
  
```shell
---
- hosts: localhost
  remote_user: root
  roles:
    - ansible-role-elasticsearch

```
</details>

***

### vars file (main.yml)

This configuration defines a Elasticsearch version.


<details>
<summary> Click here to see main.yml file</summary>
<br>
  
```shell
# vars file for ansible-role-elasticsearch
elasticsearch_version: "7.x"
```
</details>

***

# Output

### Elasticsearch Role Execution
![Screenshot from 2024-03-30 15-38-34](https://github.com/CodeOps-Hub/Ansible/assets/156056746/4a4ab6d2-f59b-4fe1-b92b-e0c4e49c1e86)


***

# Conclusion
In conclusion, this Ansible role simplifies the deployment and management of Elasticsearch clusters. It automates the installation and configuration process, reducing manual effort and ensuring consistency across environments.
***

# Contact Information

| **Name** | **Email Address** |
| -------- | ----------------- |
| **Shikha Tripathi** | shikha.tripathi.snaatak@mygurukulam.co |

***

# References

| **Source** | **Description** |
| ---------- | --------------- |
| [Link](https://docs.ansible.com/ansible/latest/index.html) | Ansible Doc Link. |
| [Link](https://www.elastic.co/guide/en/elasticsearch/reference/current/targz.html) | Elasticserach Setup. |
| [Link](https://www.youtube.com/watch?v=junPdh2yvbU&t=454s) | Reference Link For Ansible Dynamic Inventory. |

