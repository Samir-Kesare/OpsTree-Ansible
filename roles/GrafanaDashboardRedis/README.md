# Ansible role for Redis Grafana Dashboard

|   Author        |  Created on   |  Version   | Last updated by  | Last edited on |
| --------------- | --------------| -----------|----------------- | -------------- |
| Khushi Malhotra |   14 April 2024  |  Version 1 | Khushi Malhotra  | 14 April 2024    |

## Introduction
This role is designed to automate the installation and configuration of Setting Redis Grafana Dashboard on target ubuntu server(Grafana Server). This role aims to simplify the process of setting up Grafana Dashboard on Grafana Server.

## Flow Diagram

![image](https://github.com/CodeOps-Hub/Ansible/assets/156056460/653aa230-410a-4cef-b2aa-8b6e8009a5d7)

## Pre-requisites
| Pre-requisite       | Description                                                                                                          |
|----------------------|----------------------------------------------------------------------------------------------------------------------|
| Ansible              | Ansible must be installed on the control machine from which you plan to run the playbook.                            |
| AWS CLI              | AWS CLI for providing AWS credentials to fetch resources.                                                            |
| Python               | Ensure that Python is installed on your system. Ansible relies on Python for its execution, and dynamic inventory scripts are typically written in Python.    |
| PIP (Python Package Installer) | Install pip if it's not already installed. pip is a package manager for Python that allows you to install and manage Python packages. |
| Boto3                | If your dynamic inventory script relies on AWS APIs to fetch inventory data, you'll need to install boto3 using pip. |
| Grafana Server        | Must have installed grafana with a security group having necessary ports allowed on it (22, 9090 for Prometheus, 9100 for Node Exporter, 6379 for Redis).     |

***

## Directory Structure

![image](https://github.com/CodeOps-Hub/Ansible/assets/156056460/5358d387-5e7f-4b69-a6af-0c2ff09943a2)

***

## Dynamic Inventory Setup

### ansible.cfg file

<details>
<summary> Click here to see ansible.cfg file</summary>
<br>

  ```shell
[defaults]

# some basic default values...

inventory           = aws_ec2.yml
private_key_file    = attendance_api.pem
remote_user         = ubuntu
host_key_checking = false

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
  - ap-south-1
hostnames:
  - ip-address
include_filters:
 - tag:Name:
     - 'grafana_server'
```
</details>

### playbook.yml file

<details>
<summary> Click here to see playbook.yml file</summary>
<br>
  
```shell
---
- name: Setup Grafana dashboard on Redis server
  hosts: all
  become: yes
  roles:
    - GrafanaDashboardRedis
```

</details>

***

## Dashboard Role

### tasks file (main.yml)

This Ansible playbook includes all the required tasks files.

<details>
<summary> Click here to see main.yml file</summary>
<br>
  
```shell
---
# tasks file for grafana_redis_dashboard
# tasks/main.yml
- name: Import json file template
  ansible.builtin.template:
    src: "{{ json_file_template_src }}"
    dest: "{{ json_file_dest }}"

- name: Import Redis Grafana dashboard
  community.grafana.grafana_dashboard:
    grafana_url: "{{ grafana_url }}"
    grafana_api_key: "{{ grafana_api_key }}"
    state: present
    commit_message: "Updated by {{ editor_name }}"
    overwrite: true
    path: "{{ json_file_dest }}"

- name: Remove json file (delete file)
  ansible.builtin.file:
    path: "{{ json_file_dest }}"
    state: absent
```
</details>

***

### templates file (grafana_dashboard.json.j2)

<details>
<summary> Click here to see grafana_dashboard.json.j2 file</summary>
<br>

```shell

{
  "dashboard": {
    "id": null,
    "title": "Redis Dashboard",
    "panels": [
      {
        "id": 1,
        "type": "graph",
        "title": "Memory Usage",
        "targets": [
          {
            "type": "timeseries",
            "datasource": "Redis",
            "query": "SELECT used_memory FROM redis_metrics"
          }
        ],
        "yaxes": [
          {
            "format": "bytes",
            "label": "Memory Usage"
          },
          {
            "format": "short",
            "label": "Count"
          }
        ],
        "gridPos": {
          "x": 0,
          "y": 0,
          "w": 12,
          "h": 6
        }
      },
      {
        "id": 2,
        "type": "graph",
        "title": "Connections",
        "targets": [
          {
            "type": "timeseries",
            "datasource": "Redis",
            "query": "SELECT connected_clients FROM redis_metrics"
          }
        ],
        "yaxes": [
          {
            "format": "short",
            "label": "Connections"
          },
          {
            "format": "short",
            "label": "Count"
          }
        ],
        "gridPos": {
          "x": 0,
          "y": 6,
          "w": 12,
          "h": 6
        }
      },
      {
        "id": 3,
        "type": "graph",
        "title": "Commands Processed",
        "targets": [
          {
            "type": "timeseries",
            "datasource": "Redis",
            "query": "SELECT total_commands_processed FROM redis_metrics"
          }
        ],
        "yaxes": [
          {
            "format": "short",
            "label": "Commands Processed"
          },
          {
            "format": "short",
            "label": "Count"
          }
        ],
        "gridPos": {
          "x": 12,
          "y": 0,
          "w": 12,
          "h": 6
        }
      },
      {
        "id": 4,
        "type": "graph",
        "title": "Evicted Keys",
        "targets": [
          {
            "type": "timeseries",
            "datasource": "Redis",
            "query": "SELECT evicted_keys FROM redis_metrics"
          }
        ],
        "yaxes": [
          {
            "format": "short",
            "label": "Evicted Keys"
          },
          {
            "format": "short",
            "label": "Count"
          }
        ],
        "gridPos": {
          "x": 12,
          "y": 6,
          "w": 12,
          "h": 6
        }
      }
    ],
    "refresh": "10s",
    "schemaVersion": 22,
    "timezone": "browser",
    "version": 1
  },
  "folderId": null,
  "overwrite": false
}
```
</details>

***

### handlers file (main.yml)

<details>
<summary> Click here to see main.yml file</summary>
<br>

```shell
---
# handlers file for grafana_redis_dashboard
# handlers/main.yml
- name: Restart Grafana service
  service:
    name: grafana-server
    state: restarted

```
</details>

***

### vars file (main.yml)

<details>
<summary> Click here to see main.yml file</summary>
<br>

```shell

---
# vars file for grafana_redis_dashboard
editor_name : "Khushi"
grafana_api_key: "glsa_XsqkwpCDXIuE5yohe7iphViJKncgEMeC_db725417"
grafana_url: http://3.110.132.100:3000
json_file_template_src: "grafana_dashboard.json.j2"
json_file_dest: "/home/ubuntu/grafana_dashboard.json.j2"

#  template
Redisy_DS_uid : "fdi6x7lbmqxvkf"
dashboard_name: "Redis-API"
dashboard_uid: "007"
````
</details>

***

## Output

### Role Execution
![GDR1](https://github.com/CodeOps-Hub/Ansible/assets/156056460/4abbabe6-24f7-41fd-aa66-97e0a5755099)

### Grafana Dashboard Execution
![GDR2](https://github.com/CodeOps-Hub/Ansible/assets/156056460/36f4e4bb-f30c-4b5e-aab9-c9884de7196a)
![GDR3](https://github.com/CodeOps-Hub/Ansible/assets/156056460/d4a3a304-8a11-4efb-a103-2b83b60d009c)

***
# Conclusion

This guide illustrates the process of deploying **Grafana Dashboard for Redis** through Ansible. By adhering to these instructions, you can effectively provision and set up `Grafana Dashboard Redis` within your AWS infrastructure.

## Contact Information
| Name            | Email Address                        |
|-----------------|--------------------------------------|
| Khushi Malhotra | khushi.malhotra.snaatak@mygurukulam.co |
***

## References

| Description                                   | References  |
| --------------------------------------------  | -------------------------------------------------|
|  Ansible Doc | [Reference link](https://docs.ansible.com/ansible/latest/index.html) |
| Reference Link For Ansible Dynamic Inventory | [Reference link](https://devopscube.com/setup-ansible-aws-dynamic-inventory/) |
