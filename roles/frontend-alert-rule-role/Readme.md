
# Alert Rules Ansible Role For Frontend Application 

<img width="500" alt="image" src="https://github.com/CodeOps-Hub/Ansible/assets/156057205/a391e3c9-1bad-4079-bbd8-dd5ce918abdc">


***

|   Author     |  Created on   |  Version   | Last updated by | Last edited on |
| ------------ | --------------| -----------|---------------- |--------------- |
| Vikram BISHT | 07 Apr 2024   |     v1     | Vikram Bisht    | 08 Apr 2024    |

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

This role is designed to automate the configuration of `Alert Rules` for `Salary APP` on target ubuntu server `prometheus`. The "Alert Rule Role for Salary App" plays a crucial role in managing and monitoring the performance and health of the Salary App infrastructure.By integrating custom alert rules into Prometheus, this role ensures proactive identification and resolution of potential issues, thereby enhancing the reliability and availability of the Salary App.

***

# Flow Diagram

<img width="900" alt="image" src="https://github.com/CodeOps-Hub/Ansible/assets/156057205/e636806a-9cfb-439e-80a4-a43a56197929">

***

# Pre-requisites

| **Pre-requisite** | **Description** |
| ----------------- | --------------- |
| **Ansible**       | Ansible must be installed on the control machine from which you plan to run the playbook. |
| **AWS CLI**       | AWS CLI for providing aws credentials to fetch resources. |
| **Python**        | Ensure that Python is installed on your system. Ansible relies on Python for its execution, and dynamic inventory scripts are typically written in Python. |
| **PIP (Python Package Installer)** | Install pip if it's not already installed. pip is a package manager for Python that allows you to install and manage Python packages. |
| **Boto3**   |  If your dynamic inventory script relies on AWS APIs to fetch inventory data, you'll need to install `boto3` using `pip`. |
| **Configured Salary API** | Must have configured Salary API with it's pre-requisites (`scylla` & `redis`). |
| **Prometheus Server** | Configured Prometheus Server to monitor metrics. |


***

# Directory Structure

<img width="500" alt="image" src="https://github.com/CodeOps-Hub/Ansible/assets/156057205/f9401f2a-1d5a-4e41-91c4-a68a7c6c32d2">

***

## Dynamic Inventory Setup

### ansible.cfg file

<details>
<summary> Click here to see ansible.cfg file</summary>
<br>
  
```shell
[defaults]
roles_path=alert_rule
retry_files_enabled=no
inventory=aws_ec2.yml
host_key_checking = False
remote_user = ubuntu
private_key_file = snaatak.pem
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
    - alert-rule

```
</details>

***

## Alert Rule Role For Salary APP

### defaults file (main.yml)

The defaults/main.yml file helps maintain consistency and simplifies role configuration by defining default values for variables. 

<details>
<summary> Click here to see main.yml file</summary>
<br>

```shell  
---
path: /home/prometheus/prometheus-2.47.1.linux-amd64/prometheus.yml
salary_alert_rules_path: /home/prometheus/prometheus-2.47.1.linux-amd64/salary_alert.rules.yml

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
  - name: salary_api_alerts
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
        expr: rate(http_server_requests_seconds_sum{job="salary_api"}[5m]) > 1
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
    src: salary_alert.rules.yml
    dest: "{{ salary_alert_rules_path }}"
  notify: Reload Prometheus

- name: Ensure Prometheus alerts rules are included in Prometheus config
  lineinfile:
    path: "{{ path }}"
    line: '- salary_alert.rules.yml'
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
    - alert_rule

```
</details>

***

# Output

## Alert Rule  Role Execution

<img width="900" alt="image" src="https://github.com/CodeOps-Hub/Ansible/assets/156057205/c8b3af92-a6eb-4db9-a769-554929e9cee3">

***

## Status of Salary APP Server in Prometheus server

<img width="900" alt="image" src="https://github.com/CodeOps-Hub/Ansible/assets/156057205/73793771-7d46-4984-ae65-55ac8a5f2ca4">

***

## Monitoring configured `Alert Rules` in Prometheus server

<img width="900" alt="image" src="https://github.com/CodeOps-Hub/Ansible/assets/156057205/fccf743b-77fa-41e2-8d5b-fb45c8040fe8">

***

<img width="900" alt="image" src="https://github.com/CodeOps-Hub/Ansible/assets/156057205/21e83188-568c-4235-8c91-47e47de81bb9">

***

<img width="900" alt="image" src="https://github.com/CodeOps-Hub/Ansible/assets/156057205/6cd71cb2-b3d7-4c6d-889d-bb8cbfe2a4e4">

***

<img width="900" alt="image" src="https://github.com/CodeOps-Hub/Ansible/assets/156057205/e9b7247a-6239-4dc5-bb5c-c13182b386ed">

***

# Conclusion

In conclusion, the "Alert Rule Role for Salary App" is essential for keeping the Salary App running smoothly. It helps us watch important metrics and quickly fix any problems. By using custom alert rules with Prometheus, we can catch issues early and prevent them from causing big disruptions. With this role, we can be sure that the Salary App stays reliable and users have a great experience.

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
| [Link](https://gitlab.cern.ch/acc-logging-team/nxcals/blob/524c1d6d29f5ad705229efa2f682e31017f640c1/ansible/roles/prometheus/templates/alerts/node-general.yml) | Alert Rule Ansible Role Reference.|
