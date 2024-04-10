#  Ansible role for alerting rules of Scylla




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

Creating an Ansible role for managing alerting rules of Scylla involves automating the configuration of Prometheus alerting rules specific to Scylla monitoring. This role streamlines the process of deploying and updating alerting rules, ensuring consistency and efficiency across your infrastructure.
***

# Flow Diagram


![image](https://github.com/CodeOps-Hub/Ansible/assets/156056570/eec65990-a27b-4f48-82e8-b9990828548c)



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

![image](https://github.com/CodeOps-Hub/Ansible/assets/156056570/2c1c4eee-0b70-4099-8810-89b354ac46cd)

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
- hosts: prometheus
  become: yes
  gather_facts: yes
  roles:
    - node_exporter_redis

```
</details>

***

##  scylla_alerting_rules

### defaults

The defaults/main.yml file helps maintain consistency and simplifies role configuration by defining default values for variables. 

<details>
<summary> main.yml </summary>
<br>
  
```shell
---
# defaults file for scylla_alerting_rules

# Path to store Scylla alerting rules
scylla_alerting_rules_path: "/etc/prometheus/scylla_alerting_rules.yml"

# Path to the Prometheus configuration directory
prometheus_config_dir: "/etc/prometheus"

```
</details>

***
### handlers


<details>
<summary> main.yml </summary>
<br>
  
```shell
---
# handlers file for scylla_alerting_rules

- name: Reload Prometheus
  service:
    name: prometheus.service
    state: restarted
  become: true

```
</details>

***
### templates/scylla_alerting_rules.yml.j2


<details>
<summary> main.yml </summary>
<br>
  
```shell
groups:
  - name: database_alerts
    rules:
      - alert: HighCPUUsage
        expr: 100 - (avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 90
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High CPU Usage Detected"
          description: "CPU usage is above 90% for 5 minutes or more. Potential performance issues or resource contention."
          
      - alert: HighMemoryUsage
        expr: (node_memory_MemTotal_bytes - node_memory_MemFree_bytes) / node_memory_MemTotal_bytes * 100 > 90
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High Memory Usage Detected"
          description: "Memory usage is above 90% for 5 minutes or more. Potential memory leaks or inefficient queries."
          
      - alert: LowDiskSpace
        expr: node_filesystem_free_bytes / node_filesystem_size_bytes * 100 < 10
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Low Disk Space Detected"
          description: "Disk space is below 10% for 5 minutes or more. Disk space management required or potential disk failures."
          
      - alert: HighConnectionPooling
        expr: process_open_fds / process_max_fds * 100 > 90
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High Connection Pooling Detected"
          description: "Number of active database connections is above 90% of the maximum for 5 minutes or more. Potential connection leaks or exhaustion."
          
      - alert: SlowQueryExecution
        expr: histogram_quantile(0.95, rate(database_query_duration_seconds_bucket[5m])) > 1
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Slow Query Execution Detected"
          description: "Average execution time of database queries is above 1 second for 5 minutes or more. Potential performance degradation or inefficient queries."
          
      - alert: DatabaseReplicationLag
        expr: database_replication_lag_seconds > 60
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Database Replication Lag Detected"
          description: "Replication lag is above 60 seconds for 5 minutes or more. Potential replication issues or network latency."
          
      - alert: DatabaseBackupStatus
        expr: database_backup_last_successful_timestamp_seconds < (time() - 86400)
        labels:
          severity: critical
        annotations:
          summary: "Database Backup Status Issue Detected"
          description: "Database backup was not performed in the last 24 hours. Data integrity and disaster recovery preparedness may be compromised."
          
      - alert: DatabaseServiceAvailability
        expr: up == 0
        labels:
          severity: critical
        annotations:
          summary: "Database Service Unavailable"
          description: "Database service is unreachable or experiencing downtime. High availability and reliability are compromised."
          
      - alert: SecurityAccessViolations
        expr: security_access_violations_total > 0
        labels:
          severity: critical
        annotations:
          summary: "Security Access Violations Detected"
          description: "Unauthorized access attempts, privilege escalations, or other security violations detected. Database security may be compromised."
          
      - alert: DatabaseDeadlocks
        expr: database_deadlocks_total > 10
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Database Deadlocks Detected"
          description: "Database deadlocks occurred more than 10 times in the last 5 minutes. Potential application or database design issues."

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
- name: Create directory for Scylla alerting rules
  ansible.builtin.file:
    path: "{{ scylla_alerting_rules_path | dirname }}"
    state: directory
    mode: '0755'

- name: Copy Scylla alerting rules file
  ansible.builtin.template:
    src: templates/scylla_alerting_rules.yml.j2
    dest: "{{ scylla_alerting_rules_path }}"
    mode: '0644'
  notify: Reload Prometheus

- name: Ensure Prometheus alerts rules are included in Prometheus config
  lineinfile:
    path: "/etc/prometheus/prometheus.yml"
    line: '- scylla_alerting_rules.yml'
    insertafter: 'rule_files'
    state: present
  notify: Reload Prometheus

```
</details>

***


***

# Output

## Alert Rule Role Execution

![Screenshot 2024-04-10 191017](https://github.com/CodeOps-Hub/Ansible/assets/156056570/abceff1b-1378-4edc-b9e4-75c0a8ef8d5b)

***

## Prometheus Server Status 

![Screenshot 2024-04-10 190630](https://github.com/CodeOps-Hub/Ansible/assets/156056570/fa33a797-3064-4647-9de8-2b09857f81b6)

***
## Prometheus Server/ Rules

![Screenshot 2024-04-10 190711](https://github.com/CodeOps-Hub/Ansible/assets/156056570/d225c7fa-e2fa-4a59-833d-f7bfa101b8a4)


***
# Conclusion


In conclusion, the Ansible role for managing alerting rules of Scylla provides a structured and automated approach to configuring alerting rules for Scylla databases.
By using this Ansible role, system administrators can streamline the process of configuring alerting rules for Scylla databases, ensuring consistent monitoring and proactive response to potential issues.

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
| [Link](https://prometheus.io/docs/prometheus/latest/configuration/alerting_rules/) | Alerting Rules in Prometheus |
