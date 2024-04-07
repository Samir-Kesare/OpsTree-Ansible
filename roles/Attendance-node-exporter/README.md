# Ansible role for Attendance Node Exporter

|   Author        |  Created on   |  Version   | Last updated by  | Last edited on |
| --------------- | --------------| -----------|----------------- | -------------- |
| Khushi Malhotra |  02 April 2024  |  Version 1 | Khushi Malhotra  | 03 April 2024    |


![image](https://github.com/CodeOps-Hub/Ansible/assets/156056460/f6848ff7-85e2-4d6c-a04b-855ae561198c)

## Introduction
This role is designed to automate the installation and configuration of Node Exporter on target ubuntu server(Attendance). This role aims to simplify the process of setting up Attendance Node Exporter server.

## Flow Diagram

![image](https://github.com/CodeOps-Hub/Ansible/assets/156056460/653aa230-410a-4cef-b2aa-8b6e8009a5d7)

***

## Pre-requisites
| Pre-requisite       | Description                                                                                                          |
|----------------------|----------------------------------------------------------------------------------------------------------------------|
| Ansible              | Ansible must be installed on the control machine from which you plan to run the playbook.                            |
| AWS CLI              | AWS CLI for providing AWS credentials to fetch resources.                                                            |
| Python               | Ensure that Python is installed on your system. Ansible relies on Python for its execution, and dynamic inventory scripts are typically written in Python.    |
| PIP (Python Package Installer) | Install pip if it's not already installed. pip is a package manager for Python that allows you to install and manage Python packages. |
| Boto3                | If your dynamic inventory script relies on AWS APIs to fetch inventory data, you'll need to install boto3 using pip. |
| Attendance Server        | Must have installed Salary with a security group having necessary ports allowed on it (22, 9090 for Prometheus, 9100 for Node Exporter, 5432 for Salary).     |

***

## Directory Structure
![image](https://github.com/CodeOps-Hub/Ansible/assets/156056460/24cb082b-0778-440c-af96-b205d6b76872)
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
     - 'Attendance_API'
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
    - node_exporter
```

</details>

***

## Attendance-Node-Exporter Role

### tasks file (main.yml)

This Ansible playbook includes all the required tasks files.

<details>
<summary> Click here to see main.yml file</summary>
<br>
  
```shell

---
# tasks file for node_exporter
- name: Ensure the required packages are installed
  become: yes
  package:
    name:
      - wget
      - tar
  ignore_errors: yes

- name: Add the 'node_exporter' user
  become: yes
  user:
    name: "{{user}}"
    shell: /bin/bash
    create_home: yes
    state: present

- name: Create a directory for node_exp installation
  become: yes
  file:
    path: /home/node_exp
    owner: "{{user}}"
    group: "{{group}}"
    state: directory

- name: download node_exporter archive
  become: yes
  get_url:
    url: https://github.com/prometheus/node_exporter/releases/download/v{{ version }}/node_exporter-{{ version }}.linux-amd64.tar.gz
    dest: /home/node_exp/

- name: Extract node_exporter archive
  become: yes
  ansible.builtin.unarchive:
    src: /home/node_exp/node_exporter-{{ version }}.linux-amd64.tar.gz
    dest: /home/node_exp
    remote_src: yes
    creates: /home/node_exp/node_exporter-{{ version }}.linux-amd64
- name: create an empty file for node_exp.services
  become: yes
  ansible.builtin.file:
    path: /etc/systemd/system/node_exp.service
    state: touch
    owner: "{{user}}"
    group: "{{group}}"
    mode: 0644

- name: Replace the content of /etc/systemd/system/node_exp.service
  template:
    src: node_exp.service.j2
    dest: /etc/systemd/system/node_exp.service
    owner: root
    group: root
    mode: 0644
  become: true

- name: Reload systemd
  become: yes
  command: systemctl daemon-reload

- name: Creating a script
  copy:
    dest: ./run_node_exporter.sh
    content: |
      #!/bin/bash
      # Change to the desired directory
      cd /home/node_exp/node_exporter-1.7.0.linux-amd64
      # Run the node_exporter command in the background
      ./node_exporter &

- name: script
  become: yes
  command: cat ./run_node_exporter.sh
- name: giving permission to execute
  file:
    path: ./run_node_exporter.sh
    mode: 777


- name: running script
  become: yes
  command: ./run_node_exporter.sh


# - name: command
#   become: yes
#   command: ./node_exporter chdir=/home/node_exp/node_exporter-1.7.0.linux-amd64 &

```
</details>

***

### templates file (node_exporter_service.j2)

This configuration sets up a systemd service for Prometheus Node Exporter tailored for Attendance App monitoring. It ensures that the service starts after the network is available. The service runs as the user and group "node_exporter" and launches Node Exporter with Attendance-specific collectors enabled, listening on the specified address and port. Finally, it specifies that the service should be enabled and started during the multi-user boot sequence.

<details>
<summary> Click here to see node_exporter_service.j2 file</summary>
<br>

```shell
[Unit]
Description={{serviceName}}
Wants=network-online.target
After=network-online.target

[Service]
User={{ user }}
Group={{ group}}
Restart=on-failure
Type=simple
ExecStart={{ exec_command }}

[Install]
WantedBy=multi-user.target
```
</details>

***

<details>
<summary> Click here to see node_exporter_config.j2 file</summary>
<br>

```shell
- job_name: 'node_exporter'
    scrape_interval: 5s
    static_configs:
      - targets:
```
</details>

***

### handlers file (main.yml)

<details>
<summary> Click here to see main.yml file</summary>
<br>

```shell
---
# handlers file for node_exporter
- name: Reload systemd
  become: yes
  command: systemctl daemon-reload
  listen: systemd_reload
```

</details>

***

## Output

### Role Execution

![image](https://github.com/CodeOps-Hub/Ansible/assets/156056460/ef596bac-eff2-463a-9153-05c62b1f5fa2)

***

### Attendance-Node-Exporter-Server

![image](https://github.com/CodeOps-Hub/Ansible/assets/156056460/0cdc3897-8fce-4998-919a-cb7eda54fe28)

***

# Conclusion

This guide illustrates the process of deploying **Attendance-node-exporter** through Ansible. By adhering to these instructions, you can effectively provision and set up `node exporter in Attendance server` within your AWS infrastructure.

## Contact Information
| Name            | Email Address                        |
|-----------------|--------------------------------------|
| Khushi Malhotra | khushi.malhotra.snaatak@mygurukulam.co |
***

## References

| Description                                   | References  |
| --------------------------------------------  | -------------------------------------------------|
|  Ansible Doc | [Reference link](https://docs.ansible.com/ansible/latest/index.html) |
| Node Exporter Setup | [Reference link](https://codewizardly.com/prometheus-on-aws-ec2-part2/) |
| Reference Link For Ansible Dynamic Inventory | [Reference link](https://devopscube.com/setup-ansible-aws-dynamic-inventory/) |
| Node Exporter Releases| [Reference link](https://github.com/prometheus/node_exporter/releases) |
