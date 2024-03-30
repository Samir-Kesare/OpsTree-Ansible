
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
# vars file for ansible-role-elasticsearch
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

```
</details>

***

### tasks file (main.yml)

This playbook automates the setup process, ensuring a smooth installation and configuration of Prometheus on the target system.

<details>
<summary> Click here to see main.yml file</summary>
<br>
  
```shell
- name: Ensure the required packages are installed
  become: yes
  package:
    name:
      - wget
      - tar
  ignore_errors: yes

- name: Add the 'prometheus' user
  become: yes
  user:
    name: "{{user}}"
    shell: /bin/bash
    create_home: yes
    state: present

- name: Create a directory for Prometheus installation
  become: yes
  file:
    path: /home/prometheus
    owner: "{{user}}"
    group: "{{group}}"
    state: directory

- name: Download Prometheus archive
  become: yes
  get_url:
    url: "{{url}}"
    dest: "{{dir}}"

- name: Extract Prometheus archive
  become: yes
  ansible.builtin.unarchive:
    src: "{{dir}}"
    dest: /home/prometheus
    remote_src: yes
    creates: "{{create_dir}}"

- name: create an empty file for prometheus.service
  ansible.builtin.file:
    path: "{{path}}"
    state: touch
    owner: "{{user}}"
    group: "{{group}}"
    mode: 0644

- name: Replace the content of "{{content_dest}}"
  become: yes  
  template:
    src: prometheus_config.j2
    dest: "{{content_dest}}"
    owner: "{{user}}"
    group: "{{group}}"
    mode: 0644

- name: Replace the content of "{{path}}"
  template:
    src: prometheus_service.j2
    dest: "{{path}}"
    owner: root
    group: root
    mode: 0644
  become: true
  notify: systemd_reload
- meta: flush_handlers

- name: Reload systemd to apply changes
  service:
    name: "{{user}}.service"
    state: restarted
  become: true

```
</details>

***

### templates file (prometheus_config.j2)

This configuration file is for setting up and managing the Prometheus server, defines the behavior of the Prometheus service, including how it should be started, restarted.

<details>
<summary> Click here to see prometheus_config.j2 file</summary>
<br>
  
```shell
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["54.236.39.26:9090"]

```
</details>

***

### templates file (prometheus_service.j2)

<details>
<summary> Click here to see prometheus_service.j2 file</summary>
<br>
  
```shell
[Unit]
Description=Prometheus Server
Documentation=https://prometheus.io/docs/introduction/overview/
After=network-online.target

[Service]
user=root
#User=prometheus
#Group=prometheus
Restart=on-failure 
ExecStart={{ exec_command }}

[Install]
WantedBy=multi-user.target

```
</details>

***

### tests file (test.yml)

This Ansible playbook named "Install prometheus" targets the localhost machine and utilizes the "Prometheus_role" role to install Prometheus.

<details>
<summary> Click here to see test.yml file</summary>
<br>
  
```shell
---
- name: Install prometheus
  hosts: localhost
  become: yes
  roles:
    - Prometheus_role

```
</details>

***

### vars file (main.yml)

This configuration defines a user and group named "prometheus" and specifies an executable command to run Prometheus. It also includes details such as the version, download URL, directory paths, and service file path. Additionally, it creates a systemd service for Prometheus and specifies the destination for the Prometheus configuration file. 

<details>
<summary> Click here to see main.yml file</summary>
<br>
  
```shell

user: "prometheus"
group: "prometheus"
exec_command: "/home/prometheus/prometheus-2.47.1.linux-amd64/prometheus --config.file=/home/prometheus/prometheus-2.47.1.linux-amd64/prometheus.yml"
version: "2.47.1"
url: "https://github.com/prometheus/prometheus/releases/download/v2.47.1/prometheus-2.47.1.linux-amd64.tar.gz"
dir: "/home/prometheus/prometheus-2.47.1.linux-amd64.tar.gz"
create_dir: "/home/prometheus/prometheus-2.47.1.linux-amd64"
path: "/etc/systemd/system/prometheus.service"
content_dest: "/home/prometheus/prometheus-2.47.1.linux-amd64/prometheus.yml"

```
</details>

***

# Output

### Prometheus Role Execution

<img width="900" alt="image" src="https://github.com/CodeOps-Hub/Ansible/assets/156057205/833e5131-1a51-430c-b2f3-2fd997915493">

***

### Prometheus Server 

<img width="900" alt="image" src="https://github.com/CodeOps-Hub/Ansible/assets/156057205/788ccce1-049b-47ba-8d04-b573066cfd06">

***

# Conclusion

This guide illustrates the process of deploying **prometheus** through Ansible. By adhering to these instructions, you can effectively provision and set up `prometheus` within your AWS infrastructure.

***

# Contact Information

| **Name** | **Email Address** |
| -------- | ----------------- |
| **Shreya Jaiswal** | shreya.jaiswal.snaatak@mygurukulam.co |

***

# References

| **Source** | **Description** |
| ---------- | --------------- |
| [Link](https://docs.ansible.com/ansible/latest/index.html) | Ansible Doc Link. |
| [Link](https://faun.pub/setting-up-prometheus-server-with-ansible-ac1f14548bce) | Prometheus Setup. |
| [Link](https://www.youtube.com/watch?v=junPdh2yvbU&t=454s) | Reference Link For Ansible Dynamic Inventory. |

