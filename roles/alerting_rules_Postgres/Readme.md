
# Alert Rules Ansible Role For Frontend Application 

![image](https://github.com/CodeOps-Hub/Ansible/assets/79625874/a388b332-90fb-4745-8533-604dbf0d5c34)


***

|   Author     |  Created on   |  Version   | Last updated by | Last edited on |
| ------------ | --------------| -----------|---------------- |--------------- |
| Vikram BISHT | 12 Apr 2024   |     v1     | Vikram Bisht    | 15 Apr 2024    |

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

This role is designed to automate the configuration of `Alert Rules` for `Postgres` on target ubuntu server `prometheus`. The "Alert Rule Role for Postgres" plays a crucial role in managing and monitoring the performance and health of the Database infrastructure.By integrating custom alert rules into Prometheus, this role ensures proactive identification and resolution of potential issues, thereby enhancing the reliability and availability of the Postgres application.

***

# Flow Diagram

![image](https://github.com/CodeOps-Hub/Ansible/assets/79625874/798b8dcc-0789-4287-8059-6287ba30cae2)

***

# Pre-requisites

| **Pre-requisite** | **Description** |
| ----------------- | --------------- |
| **Ansible**       | Ansible must be installed on the control machine from which you plan to run the playbook. |
| **AWS CLI**       | AWS CLI for providing aws credentials to fetch resources. |
| **Python**        | Ensure that Python is installed on your system. Ansible relies on Python for its execution, and dynamic inventory scripts are typically written in Python. |
| **PIP (Python Package Installer)** | Install pip if it's not already installed. pip is a package manager for Python that allows you to install and manage Python packages. |
| **Boto3**   |  If your dynamic inventory script relies on AWS APIs to fetch inventory data, you'll need to install `boto3` using `pip`. |
| **Prometheus Server** | Configured Prometheus Server to monitor metrics. |


***

# Directory Structure

![image](https://github.com/CodeOps-Hub/Ansible/assets/79625874/60eb2521-2521-407a-a6db-2cc30fd962f1)

***

## Dynamic Inventory Setup

### ansible.cfg file

<details>
<summary> Click here to see ansible.cfg file</summary>
<br>
  
```shell
[defaults]

inventory            = aws_ec2.yml
host_key_checking    = False
private_key_file     = snaatak.pem
remote_user          = ubuntu

[inventory]
enable_plugins       = aws_ec2

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
    - prometheus

```
</details>

***

### playbook.yml file

<details>
<summary> Click here to see playbook.yml file</summary>
<br>
  
```shell
---
- hosts: prometheus
  become: yes
  gather_facts: yes
  roles:
    - alerting_rule_Postgres

```
</details>

***

## Alert Rule Role For Postgres Database

### defaults file (main.yml)

The defaults/main.yml file helps maintain consistency and simplifies role configuration by defining default values for variables. 

<details>
<summary> Click here to see main.yml file</summary>
<br>

```shell  
---
path: /home/prometheus/prometheus-2.47.1.linux-amd64/prometheus.yml
frontend_alert_rules_path: /home/prometheus/prometheus-2.47.1.linux-amd64/alert_rules.yml

```
</details>

***

### handlers file (main.yml)

This Ansible task named "Reload Prometheus" executes the command systemctl prometheus reload to restart the`prometheus.service`. It includes a start directive with the value `restarted`, indicating that other tasks can notify this task to trigger a restart of `prometheus.service`. 

<details>
<summary> Click here to see main.yml file</summary>
<br>
  
```shell
- name: Reload Prometheus
  service:
    name: prometheus.service
    state: restarted
  become: true

```
</details>

***

### files/main.yml file

<details>
<summary> Click here to see main.yml file</summary>
<br>

```shell  
---
groups:
  - name: frontend_api_alerts
    rules:
      - alert: HighCPULoad
        expr: system_cpu_usage > 90
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: High CPU Load
          description: 'The CPU load on the server is high.'
      
      - alert: MemoryUsageHigh
        expr: jvm_memory_used_bytes / jvm_memory_max_bytes * 100 > 80
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: High Memory Usage
          description: 'The memory usage on the server is high.'
          
      - alert: DiskSpaceLow
        expr: disk_free_bytes / disk_total_bytes * 100 < 10
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: Low Disk Space
          description: 'The available disk space on the server is critically low.'
          
      - alert: HighRequestLatency
        expr: rate(http_server_requests_seconds_sum{job="frontend_api"}[5m]) > 1
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: High Request Latency
          description: 'The average request latency is higher than normal.'

```
</details>


### tasks file (main.yml)

This playbook automates the setup process, ensuring a smooth installation and configuration of Prometheus on the target system.

<details>
<summary> Click here to see main.yml file</summary>
<br>
  
```shell
---
- name: Copy alert rules file to Prometheus server
  copy:
    src: alert_rules.yml
    dest: "{{ frontend_alert_rules_path }}"
  notify: Reload Prometheus

- name: Ensure Prometheus alerts rules are included in Prometheus config
  lineinfile:
    path: "{{ path }}"
    line: '- alert_rules.yml'
    insertafter: 'rule_files'
    state: present
  notify: Reload Prometheus

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
- name: Alert Rule
  hosts: localhost
  become: yes
  roles:
    - frontend-alert-rule-role

```
</details>

***

# Output

## Alert Rule  Role Execution

![image](https://github.com/CodeOps-Hub/Ansible/assets/79625874/12ce7d97-9a07-4d4e-956e-786b4e864b7c)


***

## Status of frontend application Server in Prometheus server

![image](https://github.com/CodeOps-Hub/Ansible/assets/79625874/19433338-adac-4edd-b998-4899305ad848)

***

## Monitoring configured `Alert Rules` in Prometheus server

![image](https://github.com/CodeOps-Hub/Ansible/assets/79625874/83fe47bf-f7e5-4c20-af4e-b4f2c7f1f140)

***

![image](https://github.com/CodeOps-Hub/Ansible/assets/79625874/7782e718-87f5-4d99-8fce-6b4250cc304c)

***

# Conclusion

In conclusion, the "Alert Rule Role for frontend application" is essential for keeping the frontend application running smoothly. It helps us watch important metrics and quickly fix any problems. By using custom alert rules with Prometheus, we can catch issues early and prevent them from causing big disruptions. With this role, we can be sure that the frontend application stays reliable and users have a great experience.

***

# Contact Information

|  Name                     |        	Email Address           |
| ------------              | --------------------------------|
| Vikram Bisht              |  Vikram.Bisht@opstree.com       |  

***

# References

| **Source** | **Description** |
| ---------- | --------------- |
| [Link](https://docs.ansible.com/ansible/latest/index.html) | Ansible Doc Link. |
| [Link](https://faun.pub/setting-up-prometheus-server-with-ansible-ac1f14548bce) | Prometheus Setup. |
| [Link](https://www.youtube.com/watch?v=junPdh2yvbU&t=454s) | Reference Link For Ansible Dynamic Inventory. |
| [Link](https://gitlab.cern.ch/acc-logging-team/nxcals/blob/524c1d6d29f5ad705229efa2f682e31017f640c1/ansible/roles/prometheus/templates/alerts/node-general.yml) | Alert Rule Ansible Role Reference.|
