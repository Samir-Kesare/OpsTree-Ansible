#  Ansible Role for Grafana

![image](https://github.com/CodeOps-Hub/Ansible/assets/156056364/b5a41ec2-406f-4999-a25e-a365fbfa959f)

***

| **Author** | **Created on** | **Last Updated** | **Document Version** |
| ---------- | -------------- | ---------------- | -------------------- |
| **Shantanu** | **29 March 2024** | **1 April 2024** | **v1** |

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

This role is crafted to automate both the installation and configuration of Grafana on designated Ubuntu servers, with the goal of streamlining the setup process for Grafana.
***

# Flow Diagram

![Screenshot 2024-04-01 at 1 29 13 AM](https://github.com/CodeOps-Hub/Ansible/assets/156056364/d96cecf0-a5a6-4c64-b4bd-cd766edf58bf)

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

![Screenshot 2024-04-01 at 1 11 15 AM](https://github.com/CodeOps-Hub/Ansible/assets/156056364/170b5332-9aca-4950-aa1a-7e19ae53727e)
***

# Dynamic Inventory Setup

### ansible.cfg

<details>
<summary> ansible.cfg </summary>
<br>
  
```shell
[defaults]
roles_path=grafana
retry_files_enabled=no
inventory=aws_ec2.yml
host_key_checking = False
remote_user = ubuntu
private_key_file = new.pem
[inventory]
enable_plugins = aws_ec2

```
</details>

***

### aws_ec2.yml

<details>
<summary> aws_ec2.yml </summary>
<br>
  
```shell
---
plugin: aws_ec2
regions:
  - ap-southeast-1
hostnames:
  - ip-address
filters:
  tag:Name:
    - grafana-server

```
</details>

***

### playbook.yml

<details>
<summary> playbook.yml </summary>
<br>
  
```shell
---
- hosts: aws_ec2
  become: yes
  gather_facts: yes
  roles:
    - grafana

```
</details>

***

## Grafana Role

### defaults

The defaults/main.yml file helps maintain consistency and simplifies role configuration by defining default values for variables. 

<details>
<summary> main.yml </summary>
<br>
  
```shell
---
# defaults file for roles/grafana
grafana_admin_password: "abc1234"

```
</details>

***

### tasks

This playbook automates the setup process, ensuring a smooth installation and configuration of Prometheus on the target system.

<details>
<summary> Click here to see main.yml file</summary>
<br>
  
```shell
---
# tasks file for roles/grafana
- name: install gpg
  apt:
    name: gnupg,software-properties-common
    state: present
    update_cache: yes
    cache_valid_time: 3600
- name: add gpg hey
  apt_key:
    url: "https://packages.grafana.com/gpg.key"
    validate_certs: no
- name: add repository
  apt_repository:
    repo: "deb https://packages.grafana.com/oss/deb stable main"             
    state: present
    validate_certs: no
- name: install grafana
  apt:
    name: grafana
    state: latest
    update_cache: yes
    cache_valid_time: 3600
- name: start service grafana-server
  systemd:
    name: grafana-server
    state: started
    enabled: yes
- name: wait for service up
  uri:
    url: "http://127.0.0.1:3000"
    status_code: 200
  register: __result
  until: __result.status == 200
  retries: 120
  delay: 1
- name: change admin password for grafana gui
  shell : "grafana-cli admin reset-admin-password {{ grafana_admin_password }}"
  register: __command_admin
  changed_when: __command_admin.rc !=0

```
</details>

***


***

# Output

### Grafana Role Execution
![Screenshot 2024-04-01 at 1 22 06 AM](https://github.com/CodeOps-Hub/Ansible/assets/156056364/b6b77bf5-6940-40d0-b447-ef776e7e6634)

***

### Grafana Dashboard 
![Screenshot 2024-04-01 at 1 23 14 AM](https://github.com/CodeOps-Hub/Ansible/assets/156056364/36f6f1c6-dacf-49cc-9b9b-b4de656ebcf1)

***

# Conclusion

This guide demonstrates how to deploy Grafana using Ansible. By following these instructions, you can efficiently provision and configure Grafana within your AWS infrastructure.

***

# Contact Information

| **Name** | **Email Address** |
| -------- | ----------------- |
| **Shantanu** | shantanu.chauhan.snaatak@mygurukulam.co |

***

# References

| **Source** | **Description** |
| ---------- | --------------- |
| [Link](https://docs.ansible.com/ansible/latest/index.html) | Ansible Doc Link. |
| [Link](https://www.youtube.com/watch?v=junPdh2yvbU&t=454s) | Reference Link For Ansible Dynamic Inventory. |
