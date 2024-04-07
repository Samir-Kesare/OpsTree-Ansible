# Ansible role for Redis Alerting Rules

|   Author        |  Created on   |  Version   | Last updated by  | Last edited on |
| --------------- | --------------| -----------|----------------- | -------------- |
| Khushi Malhotra |  08 April 2024  |  Version 1 | Khushi Malhotra  | 08 April 2024    |

## Introduction
This role is designed to automate the installation and configuration of Setting Alerting rules on target ubuntu server(Redis DB). This role aims to simplify the process of setting up Allerting Rules on Redis Database server.

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
| Redis Server        | Must have installed Redis with a security group having necessary ports allowed on it (22, 9090 for Prometheus, 9100 for Node Exporter, 6379 for Redis).     |

***

## Directory Structure

![image](https://github.com/CodeOps-Hub/Ansible/assets/156056460/b25e326f-1048-4087-baaf-c9c15a068776)
***

> [!NOTE]
>Ensure that for dynamic inventory you have the necessary AWS credentials configured in AWS CLI or an IAM role on the node.
> Also Refer this [*Prometheus Documentation*](https://github.com/CodeOps-Hub/Ansible/blob/shreya/prometheus-role/roles/prometheus/README.md) for `Prometheus` guide, as for configuring node exporter, it is necessary to have the functionality of fetching the servers for which you need to see logs in `Prometheus UI`.

## Dynamic Inventory Setup

### ansible.cfg file

<details>
<summary> Click here to see ansible.cfg file</summary>
<br>

  ```shell
[defaults]

# some basic default values...

inventory           = aws_ec2.yml
private_key_file    = terragrunt_cred.pem
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
     - 'Redis_DB'
```
</details>

### playbook.yml file

<details>
<summary> Click here to see playbook.yml file</summary>
<br>
  
```shell
---
- name: Playbook to apply roles
  hosts: all
  become: yes

  roles:
    - Redis_alertingrules
```

</details>

***

## Alerting Rules Role

### tasks file (main.yml)

This Ansible playbook includes all the required tasks files.

<details>
<summary> Click here to see main.yml file</summary>
<br>
  
```shell
---
# tasks file for alertingrules
# tasks file for prometheus
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
  become: true
  notify: systemd_reload


- name: Replace the content of "{{path}}"
  template:
    src: prometheus_service.j2
    dest: "{{path}}"
    owner: root
    group: root
    mode: 0644
  become: true
  notify: systemd_reload

- name: Replace the content of "{{ prometheus_rules_dest }}"
  template:
    src: first_rules.j2
    dest: "{{ prometheus_rules_dest }}"
    owner: root
    group: root
    mode: 0644
  become: true
  notify: systemd_reload

- name: Reload systemd to apply changes
  service:
    name: "{{user}}.service"
    state: started
  become: true
```
</details>

***

### templates file (first_rules.j2)
<details>
<summary> Click here to see first_rules.j2 file</summary>
<br>

```shell
{% raw %}
groups:
  - name: redis.alert.rules
    rules:
      - alert: RedisHighMemoryUsage
        expr: (redis_memory_used_bytes / redis_memory_max_bytes) * 100 > 80
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Redis high memory usage (instance {{ .Labels.instance }})"
          description: "Memory usage is above 80%\n  VALUE = {{ $value }}\n  LABELS: {{ .Labels }}"

      - alert: RedisHighConnections
        expr: redis_connected_clients > 1000
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High number of connections to Redis (instance {{ .Labels.instance }})"
          description: "Number of connected clients is greater than 1000\n  VALUE = {{ $value }}\n  LABELS: {{ .Labels }}"

      - alert: RedisReplicationLag
        expr: redis_replication_delay_seconds > 60
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Redis replication lag (instance {{ .Labels.instance }})"
          description: "Replication lag is greater than 60 seconds\n  VALUE = {{ $value }}\n  LABELS: {{ .Labels }}"
{% endraw %}
```
</details>

***

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
          - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
   - "first_rules.j2"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["44.203.178.103:9090", "localhost:9090
```
</details>

***

<details>
<summary> Click here to see prometheus_service.j2 file</summary>
<br>

  ```shell
[Unit]
Description=Prometheus Server
Documentation=https://prometheus.io/docs/introduction/overview/
After=network-online.target

[Service]
#user=root
User=prometheus
Group=prometheus
Restart=on-failure
ExecStart={{ exec_command }}
ExecReload=/bin/kill -USR1 $MAINPID
Restart=always
RestartSec=1

[Install]
WantedBy=multi-user.target
```
</details>

***
### handlers file (main.yml)

<details>
<summary> Click here to see main.yml file</summary>
<br>

```shell
---
# handlers file for alertingrules
- name: systemd_reload
  systemd:
    name: "{{ user }}.service"
    daemon_reload: yes
```
</details>
***

### vars file (main.yml)

<details>
<summary> Click here to see main.yml file</summary>
<br>

```shell
---
# vars file for alertingrules
user: "prometheus"
group: "prometheus"
exec_command: "/home/prometheus/prometheus-2.47.1.linux-amd64/prometheus --config.file=/home/prometheus/prometheus-2.47.1.linux-amd64/prometheus.yml"
version: "2.47.1"
url: "https://github.com/prometheus/prometheus/releases/download/v2.47.1/prometheus-2.47.1.linux-amd64.tar.gz"
dir: "/home/prometheus/prometheus-2.47.1.linux-amd64.tar.gz"
create_dir: "/home/prometheus/prometheus-2.47.1.linux-amd64"
path: "/etc/systemd/system/prometheus.service"
content_dest: "/home/prometheus/prometheus-2.47.1.linux-amd64/prometheus.yml"
prometheus_rules_dest: "/home/prometheus/prometheus-2.47.1.linux-amd64/first_rules.j2"
```
</details>
***

<details>
<summary> Click here to see redis_vars.yml file</summary>
<br>

```shell
# vars/redis_vars.yml

redis_memory_used_bytes: 500000000 # Example value for memory used in bytes
redis_memory_max_bytes: 1000000000 # Example value for maximum memory in bytes
redis_connected_clients: 800 # Example value for connected clients
redis_replication_delay_seconds: 30 # Example value for replication delay in seconds
```
</details>

***

## Output

### Role Execution
![image](https://github.com/CodeOps-Hub/Ansible/assets/156056460/8cad27a2-d540-4d62-b47c-2c8b7c4d1be6)

***
# Conclusion

This guide illustrates the process of deploying **Alerting Rules** through Ansible. By adhering to these instructions, you can effectively provision and set up `node exporter in Attendance server` within your AWS infrastructure.

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





