#  Ansible Role for Node Exporter setup of Redis

![image](https://github.com/CodeOps-Hub/Ansible/assets/156056570/197a4197-fdca-41b4-97cf-fccbecb565f1)


***

| **Author** | **Created on** | **Last Updated** | **Document Version** |
| ---------- | -------------- | ---------------- | -------------------- |
| **Samir** | **09-04-2024** | **10-04-2024** | **v1** |

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

Redis Exporter is a tool that collects and exports Redis metrics for monitoring and alerting purposes. It retrieves information from a Redis instance and exposes it in a format that can be consumed by monitoring systems like Prometheus.
***

# Flow Diagram

![image](https://github.com/CodeOps-Hub/Ansible/assets/156056570/30ff6f07-bfbd-4f62-a167-963256df414d)



***

# Pre-requisites

| **Pre-requisite** | **Description** |
| ----------------- | --------------- |
| **Ansible**       | Ansible must be installed on the control machine from which you plan to run the playbook. |
| **AWS CLI**       | AWS CLI for providing aws credentials to fetch resources. |
| **Python**        | Ensure that Python is installed on your system. Ansible relies on Python for its execution, and dynamic inventory scripts are typically written in Python. |
| **PIP (Python Package Installer)** | Install pip if it's not already installed. pip is a package manager for Python that allows you to install and manage Python packages. |
| **Boto3**   |  If your dynamic inventory script relies on AWS APIs to fetch inventory data, you'll need to install `boto3` using `pip`. |
| **Prometheus Server** | Updated Prometheus Server for moniotoring logs generated through postgres-node-exporter with security group having necessary ports allowed on it (22, 9090 for prometheus, 9100 for node exporter).|
***

# Directory Structure

![image](https://github.com/CodeOps-Hub/Ansible/assets/156056570/5f91c344-72e7-4b2d-9024-ac61f77b211c)

***

# Dynamic Inventory Setup

### ansible.cfg

<details>
<summary> ansible.cfg </summary>
<br>
  
```shell
[defaults]
# some basic default values...
inventory               =       ./aws_ec2.yaml
private_key_file        =       ./advik.pem
remote_user             =       ubuntu
host_key_checking       =       False
[inventory]
enable_plugins          =       aws_ec2
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
  - ap-northeast-1
filters:
    instance-state-code: 16
keyed_groups:
  - key: tags
    prefix: tag

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
    - node_exporter_redis

```
</details>

***

##  node_exporter_redis

### defaults

The defaults/main.yml file helps maintain consistency and simplifies role configuration by defining default values for variables. 

<details>
<summary> main.yml </summary>
<br>
  
```shell
---
# defaults file for node_exporter_redis

# Node Exporter Version
node_exporter_version: "1.2.2"

# Port for Node Exporter
node_exporter_port: 9100

# Redis Exporter Version
redis_exporter_version: "1.22.0"

# Port for Redis Exporter
redis_exporter_port: 9121

```
</details>

***
### handlers


<details>
<summary> main.yml </summary>
<br>
  
```shell
---
- name: Restart Node Exporter Service
  ansible.builtin.systemd:
    name: node_exporter
    state: restarted

- name: Restart Redis Exporter Service
  ansible.builtin.systemd:
    name: redis_exporter
    state: restarted

```
</details>

***
### templates/node_exporter.service.j2


<details>
<summary> main.yml </summary>
<br>
  
```shell
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

StartLimitIntervalSec=500
StartLimitBurst=5

[Service]
User=node_exporter
Group=node_exporter
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/node_exporter \
    --collector.logind

[Install]
WantedBy=multi-user.target

```
</details>

***
### templates/redis_exporter.service.j2
<details>
<summary> main.yml </summary>
<br>
  
```shell
[Unit]
Description=Prometheus
Documentation=https://github.com/oliver006/redis_exporter
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
User=node_exporter
Group=node_exporter
ExecReload=/bin/kill -HUP $MAINPID
ExecStart=/usr/local/bin/redis_exporter \
  --log-format=txt \
  --namespace=redis \
  --web.listen-address=:9121 \
  --web.telemetry-path=/metrics

SyslogIdentifier=redis_exporter
Restart=always

[Install]
WantedBy=multi-user.target
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
- name: Install required packages
  apt:
    name: "{{ item }}"
    state: present
  with_items:
    - wget
    - tar
    

- name: Download and install Node Exporter
  get_url:
    url: "https://github.com/prometheus/node_exporter/releases/download/v{{ node_exporter_version }}/node_exporter-{{ node_exporter_version }}.linux-amd64.tar.gz"
    dest: /tmp/node_exporter.tar.gz

- name: Extract Node Exporter archive
  ansible.builtin.unarchive:
    src: /tmp/node_exporter.tar.gz
    dest: /usr/local/bin/
    remote_src: yes

- name: Create systemd service file
  template:
    src: node_exporter.service.j2
    dest: /etc/systemd/system/node_exporter.service
  notify: Restart Node Exporter Service

- name: Allow Node Exporter port through firewall
  ansible.builtin.ufw:
    rule: allow
    port: "{{ node_exporter_port }}"
    proto: tcp

- name: Download Redis Exporter
  get_url:
    url: "https://github.com/oliver006/redis_exporter/releases/download/v{{ redis_exporter_version }}/redis_exporter-v{{ redis_exporter_version }}.linux-amd64.tar.gz"
    dest: /tmp/redis_exporter.tar.gz

- name: Extract Redis Exporter archive
  ansible.builtin.unarchive:
    src: /tmp/redis_exporter.tar.gz
    dest: /usr/local/bin/
    remote_src: yes

- name: Create Redis Exporter systemd service file
  template:
    src: redis_exporter.service.j2
    dest: /etc/systemd/system/redis_exporter.service
  notify: Restart Redis Exporter Service


- name: Allow Redis Exporter port through firewall
  ansible.builtin.ufw:
    rule: allow
    port: "{{ redis_exporter_port }}"
    proto: tcp

```
</details>

***


***

# Output

## Node Exporter Role Execution

![Screenshot 2024-04-09 233730](https://github.com/CodeOps-Hub/Ansible/assets/156056570/cca3494e-3794-4f9c-902c-a01813580053)


***

## Redis-Node-Exporter Server
![image](https://github.com/CodeOps-Hub/Ansible/assets/156056570/add3abb8-e8b0-4933-8be2-34f761e2dcb8)

***
## Monitoring Redis-Node-Exporter server in Prometheus

![Screenshot 2024-04-09 203815](https://github.com/CodeOps-Hub/Ansible/assets/156056570/ddea3409-2207-4c2e-b6e9-60fff7090c1f)

***
## Redis-Node-Exporter Metrics

![Screenshot 2024-04-09 192833](https://github.com/CodeOps-Hub/Ansible/assets/156056570/a5257509-c0c6-4780-a086-7cccceb9b38b)

***


# Conclusion


In summary, the Ansible Role for setting up Node Exporter with Redis monitoring allows you to automate the deployment and configuration of monitoring solutions for both system-level metrics (via Node Exporter) and Redis-specific metrics (via Redis Exporter) on your servers.

***

# Contact Information

| **Name** | **Email Address** |
| -------- | ----------------- |
| **Samir** | samir.kesare.snaatak@mygurukulam.co |

***

# References

| **Source** | **Description** |
| ---------- | --------------- |
| [Link](https://docs.ansible.com/ansible/latest/index.html) | Ansible Doc Link. |
| [Link](https://www.youtube.com/watch?v=junPdh2yvbU&t=454s) | Reference Link For Ansible Dynamic Inventory. |
| [Link](https://www.fosstechnix.com/monitor-redis-with-prometheus-and-grafana/) | Reference Link for Redis-Node-Exporter |
