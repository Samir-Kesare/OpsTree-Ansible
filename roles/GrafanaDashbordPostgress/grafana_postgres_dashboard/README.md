# Ansible role for Grafana Dashboard setup of Postgress

![image](https://github.com/CodeOps-Hub/Ansible/assets/156056570/469c5bb1-caea-4905-84f3-e0d8b60bceab)


***

| **Author** | **Created on** | **Last Updated** | **Document Version** |
| ---------- | -------------- | ---------------- | -------------------- |
| **Samir** | **10-04-2024** | **11-04-2024** | **v1** |

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

Creating an Ansible role for setting up Grafana dashboards for PostgreSQL involves automating the deployment and configuration of dashboards within Grafana, tailored to monitor PostgreSQL databases.
***

# Flow Diagram

![image](https://github.com/CodeOps-Hub/Ansible/assets/156056570/c9a095a6-8a81-414e-9fd6-4b9bfe5f5435)

***

# Pre-requisites

| **Pre-requisite** | **Description** |
| ----------------- | --------------- |
| **Ansible**       | Ansible must be installed on the control machine from which you plan to run the playbook. |
| **AWS CLI**       | AWS CLI for providing aws credentials to fetch resources. |
| **Python**        | Ensure that Python is installed on your system. Ansible relies on Python for its execution, and dynamic inventory scripts are typically written in Python. |
| **PIP (Python Package Installer)** | Install pip if it's not already installed. pip is a package manager for Python that allows you to install and manage Python packages. |
| **Boto3**   |  If your dynamic inventory script relies on AWS APIs to fetch inventory data, you'll need to install `boto3` using `pip`. |
| **Prometheus & Grafana Server** | Updated Prometheus Server for moniotoring logs generated through postgres-node-exporter with security group having necessary ports allowed on it (22, 9090 for prometheus, 9100 for node exporter).|
***

# Directory Structure

![image](https://github.com/CodeOps-Hub/Ansible/assets/156056570/f9ec4149-4131-4d74-a346-d19c415cc08a)

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
- hosts: grafana
  become: yes
  gather_facts: yes
  roles:
    - grafana_dashboard_postgress

```
</details>

***

##  grafana_dashboard_postgress


### handlers


<details>
<summary> main.yml </summary>
<br>
  
```shell


```
</details>

***

### templates/postgressDashboard.jason.j2
<details>
<summary> main.yml </summary>
<br>
  
```shell
{
  "dashboard": {
    "id": null,
    "title": "PostgreSQL Dashboard",
    "panels": [
      {
        "id": 1,
        "type": "graph",
        "title": "CPU Usage",
        "targets": [
          {
            "type": "timeseries",
            "datasource": "PostgreSQL",
            "query": "SELECT cpu_usage FROM postgresql_metrics"
          }
        ],
        "yaxes": [
          {
            "format": "percent",
            "label": "CPU Usage (%)"
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
        "title": "Memory Usage",
        "targets": [
          {
            "type": "timeseries",
            "datasource": "PostgreSQL",
            "query": "SELECT memory_usage FROM postgresql_metrics"
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
          "y": 6,
          "w": 12,
          "h": 6
        }
      },
      {
        "id": 3,
        "type": "graph",
        "title": "Disk I/O",
        "targets": [
          {
            "type": "timeseries",
            "datasource": "PostgreSQL",
            "query": "SELECT disk_io FROM postgresql_metrics"
          }
        ],
        "yaxes": [
          {
            "format": "bytes",
            "label": "Disk I/O"
          },
          {
            "format": "short",
            "label": "Count"
          }
        ],
        "gridPos": {
          "x": 0,
          "y": 12,
          "w": 12,
          "h": 6
        }
      },
      {
        "id": 4,
        "type": "graph",
        "title": "Query Execution Time",
        "targets": [
          {
            "type": "timeseries",
            "datasource": "PostgreSQL",
            "query": "SELECT query_execution_time FROM postgresql_metrics"
          }
        ],
        "yaxes": [
          {
            "format": "ms",
            "label": "Query Execution Time (ms)"
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
        "id": 5,
        "type": "graph",
        "title": "Active Database Connections",
        "targets": [
          {
            "type": "timeseries",
            "datasource": "PostgreSQL",
            "query": "SELECT active_connections FROM postgresql_metrics"
          }
        ],
        "yaxes": [
          {
            "format": "short",
            "label": "Active Connections"
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
      },
      {
        "id": 6,
        "type": "graph",
        "title": "Database Locks",
        "targets": [
          {
            "type": "timeseries",
            "datasource": "PostgreSQL",
            "query": "SELECT database_locks FROM postgresql_metrics"
          }
        ],
        "yaxes": [
          {
            "format": "short",
            "label": "Database Locks"
          },
          {
            "format": "short",
            "label": "Count"
          }
        ],
        "gridPos": {
          "x": 12,
          "y": 12,
          "w": 12,
          "h": 6
        }
      },
      {
        "id": 7,
        "type": "graph",
        "title": "Database Replication Lag",
        "targets": [
          {
            "type": "timeseries",
            "datasource": "PostgreSQL",
            "query": "SELECT replication_lag FROM postgresql_metrics"
          }
        ],
        "yaxes": [
          {
            "format": "s",
            "label": "Replication Lag (seconds)"
          },
          {
            "format": "short",
            "label": "Count"
          }
        ],
        "gridPos": {
          "x": 24,
          "y": 0,
          "w": 12,
          "h": 6
        }
      },
      {
        "id": 8,
        "type": "graph",
        "title": "Table Size",
        "targets": [
          {
            "type": "timeseries",
            "datasource": "PostgreSQL",
            "query": "SELECT table_size FROM postgresql_metrics"
          }
        ],
        "yaxes": [
          {
            "format": "bytes",
            "label": "Table Size"
          },
          {
            "format": "short",
            "label": "Count"
          }
        ],
        "gridPos": {
          "x": 24,
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
### tasks

This playbook automates the setup process, ensuring a smooth installation and configuration of Prometheus on the target system.

<details>
<summary> Click here to see main.yml file</summary>
<br>
  
```shell
---
# tasks file for attendanceGfDashboard

- name: Import json file template
  ansible.builtin.template:
    src: "{{ json_file_template_src }}"
    dest: "{{ json_file_dest }}"

- name: Import postgress Grafana dashboard 
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


***

# Output

## Role Execution

![image](https://github.com/CodeOps-Hub/Ansible/assets/156056570/a18f2e39-f259-4944-83dd-7c3f79f93c36)

***

## Prometheus Grafana Server
![image](https://github.com/CodeOps-Hub/Ansible/assets/156056570/1dc02f4f-e12d-4862-a180-bef1f7ee3733)


***
## PGA Server Dashboard View
![image](https://github.com/CodeOps-Hub/Ansible/assets/156056570/ee8c3a93-36c4-40a5-ad30-f57d7ffc763d)


***


# Conclusion

In conclusion, the Ansible role for setting up Grafana dashboards for PostgreSQL streamlines the process of deploying and managing monitoring configurations across your infrastructure.
With this Ansible role, system administrators can efficiently deploy and manage Grafana dashboards for PostgreSQL, ensuring consistent monitoring and visualization of database metrics across their infrastructure.


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
| [Link](https://grafana.com/blog/2022/06/06/grafana-dashboards-a-complete-guide-to-all-the-different-types-you-can-build/) | Reference for Grafana Dashboard |


