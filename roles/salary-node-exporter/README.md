# Ansible Role for Salary-Node-Exporter
![image](https://github.com/CodeOps-Hub/Ansible/assets/156056444/7307e336-40f7-483b-b4b5-03736aa58ee1)

***
|   Author        |  Created on   |  Version   | Last updated by  | Last edited on |
| --------------- | --------------| -----------|----------------- | -------------- |
| **[Harshit Singh](https://github.com/Panu-S-Harshit-Ninja-07)**    | 02 April 2024 |  Version 1 | Harshit Singh     | 02 April 2024  |
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

This role is designed to automate the installation and configuration of Node Exporter on target ubuntu server(Salary). This role aims to simplify the process of setting up Salary Node Exporter server.

***

# Flow Diagram

![image](https://github.com/CodeOps-Hub/Ansible/assets/156056444/b1800d8c-94aa-4db1-beeb-bacbfd3ddd22)

***

# Pre-requisites

| **Pre-requisite** | **Description** |
| ----------------- | --------------- |
| **Ansible**       | Ansible must be installed on the control machine from which you plan to run the playbook. |
| **AWS CLI**       | AWS CLI for providing aws credentials to fetch resources. |
| **Python**        | Ensure that Python is installed on your system. Ansible relies on Python for its execution, and dynamic inventory scripts are typically written in Python. |
| **PIP (Python Package Installer)** | Install pip if it's not already installed. pip is a package manager for Python that allows you to install and manage Python packages. |
| **Boto3**   |  If your dynamic inventory script relies on AWS APIs to fetch inventory data, you'll need to install `boto3` using `pip`. |
| **Salary Server** | Must have installed `Salary` with `security group` having necessary ports allowed on it (22, 9090 for prometheus, 9100 for node exporter, 8080 for Salary). |

***

# Directory Structure
![image](https://github.com/CodeOps-Hub/Ansible/assets/156056444/abb03ca9-3625-4a29-86d3-70468d7f4de5)

***

> [!NOTE]
>Ensure that for dynamic inventory you have the necessary AWS credentials configured in AWS CLI or an IAM role on the node.
> Also Refer this [*Prometheus Documentation*](https://github.com/CodeOps-Hub/Ansible/blob/shreya/prometheus-role/roles/prometheus/README.md) for `Prometheus` guide, as for configuring node exporter, it is necessary to have the functionality of fetching the servers for which you need to see logs in `Prometheus UI`, for this, I've configured `prometheus_config.j2` file in `templates/prometheus_config.j2` in which there is a section of service discovery for EC2 Instances, it will automatically fetch the servers of provided region and you can easily monitor the logs of `node-exporter's server`, as the `prometheus server` has an `IAM Role` attached on it with `AmazonEC2ReadOnlyAccess` policy. 

## Dynamic Inventory Setup

### ansible.cfg file

<details>
<summary> Click here to see ansible.cfg file</summary>
<br>
  
```shell
[defaults]
inventory		=	/home/harshit/aws_ec2.yml # path of inventory file
private_key_file	=	/home/harshit/Downloads/central.pem  # path of pem key 
remote_user		= 	ubuntu
host_key_checking	=	False
[inventory]
enable_plugins 		= 	aws_ec2
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
filters:
  instance-state-code: 16
keyed_groups:
  - key: tags
    prefix: tag
```
</details>

***

### playbook.yml file

<details>
<summary> Click here to see playbook.yml file</summary>
<br>
  
```shell
---
- hosts: tag_Name_demo_salary_node_exp #name of host
  become: true
  gather_facts: yes
  roles:
    - salary-node-exporter
```
</details>

***

## Salary-Node-Exporter Role

### pre-requisites file (prerequisites.yml)
This Ansible playbook consists of tasks aimed to meet pre-requisites of the Node Exporter service.
<details>
<summary> Click here to see prerequisites.yml file</summary>
<br>
  
```shell
---
    - name: Add node_exporter group
      ansible.builtin.group:
        name: node_exporter
        state: present

    - name: Add node_exporter user
      ansible.builtin.user:
        name: node_exporter
        group: node_exporter
        create_home: no

```
</details>

***
### ubuntu file (ubuntu.yml)
This Ansible playbook consists of tasks aimed at downloading, extracting, installing, and starting the Node Exporter service.
<details>
<summary> Click here to see ubuntu.yml file</summary>
<br>
  
```shell
---
# tasks/main.yml
- name: Download Node Exporter tarball
  get_url:
    url: "{{ node_exporter_url }}"
    dest: "/tmp/node_exporter.tar.gz"

- name: Extract Node Exporter tarball
  unarchive:
    src: "/tmp/node_exporter.tar.gz"
    dest: "/tmp"
    remote_src: true
    creates: "/tmp/node_exporter-{{ node_exporter_version }}.{{ node_exporter_architecture }}"

- name: Debug node exporter binary path
  debug:
    msg: "Node Exporter binary path: {{ '/tmp/node_exporter-1.7.0.linux-amd64/node_exporter' }}"

- name: Copy Node Exporter binary to /usr/local/bin
  become: true 
  copy:
    src: "/tmp/node_exporter-{{ node_exporter_version }}.{{ node_exporter_architecture }}/node_exporter"
    dest: "/usr/local/bin/node_exporter"
    remote_src: true
    mode: "0755"

- name: Clean up downloaded files
  file:
    path: "/tmp/{{ item }}"
    state: absent
  with_items:
    - "node_exporter.tar.gz"
    - "node_exporter-{{ node_exporter_version }}.{{ node_exporter_architecture }}"

- name: Copy systemd service template
  ansible.builtin.template:
    src: "{{ node_exporter_service_template_src }}"
    dest: "{{ node_exporter_service_dest }}"
  
- name: Reload systemd
  ansible.builtin.systemd:
    daemon_reload: yes

- name: Enable node-exporter service
  become: true
  systemd:
    name: node-exporter
    enabled: yes

- name: Start node-exporter service
  become: true
  systemd:
    name: node-exporter
    state: started

- name: Check status of node-exporter service
  become: true
  systemd:
    name: node-exporter
    state: started
  register: node_exporter_status

- debug:
    msg: |
      Node Exporter Status:
      ActiveState: {{ node_exporter_status.status.ActiveState }}
      Description: {{ node_exporter_status.status.Description }}
      UnitFileState: {{ node_exporter_status.status.UnitFileState }}
      TasksCurrent: {{ node_exporter_status.status.TasksCurrent }}
```
</details>

***
### tasks file (main.yml)

This Ansible playbook includes all the required tasks files.

<details>
<summary> Click here to see main.yml file</summary>
<br>
  
```shell
---
# tasks/main.yml
- name: Include tasks for Prequisites
  ansible.builtin.include_tasks: prerequisites.yml
  when: ansible_facts['distribution'] == 'Ubuntu'

- name: Include tasks for Ubuntu
  ansible.builtin.include_tasks: ubuntu.yml
  when: ansible_facts['distribution'] == 'Ubuntu'


```
</details>

***

### templates file (node_exporter_service.j2)

This configuration sets up a systemd service for Prometheus Node Exporter tailored for Salary App monitoring. It ensures that the service starts after the network is available. The service runs as the user and group "node_exporter" and launches Node Exporter with Salary-specific collectors enabled, listening on the specified address and port. Finally, it specifies that the service should be enabled and started during the multi-user boot sequence.

<details>
<summary> Click here to see node_exporter_service.j2 file</summary>
<br>
  
```shell
[Unit]
Description=Prometheus Node Exporter Service
After=network.target

[Service]
User={{ node_exporter_user }}
Group={{ node_exporter_group }}
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target

```
</details>

***

### tests file (test.yml)

This Ansible playbook named "test.yml" targets the localhost machine and utilizes the "Node_Exporter_role" role to install Node Exporter in Salary Server.

<details>
<summary> Click here to see test.yml file</summary>
<br>
  
```shell
---
- hosts: localhost
  become: true
  gather_facts: yes
  roles:
    - salary-node-exporter

```
</details>

***

### vars file (main.yml)

This YAML configuration sets parameters for the Node Exporter and Salary monitoring. It specifies the Node Exporter version, installation directory, listening address, telemetry path, and URL for downloading the Node Exporter binary.

<details>
<summary> Click here to see main.yml file</summary>
<br>
  
```shell
---

node_exporter_version: "1.7.0"
node_exporter_architecture: "linux-amd64"
node_exporter_url: "https://github.com/prometheus/node_exporter/releases/download/v{{ node_exporter_version }}/node_exporter-{{ node_exporter_version }}.{{ node_exporter_architecture }}.tar.gz"
node_exporter_service_template_src: "node_exporter_service.j2"
node_exporter_service_dest: "/etc/systemd/system/node-exporter.service"
node_exporter_user: "node_exporter"
node_exporter_group: "node_exporter"

```
</details>

***

# Output

### Role Execution
![image](https://github.com/CodeOps-Hub/Ansible/assets/156056444/b4cea04e-de9f-44e0-a00e-76ba22604524)

***

### Salary-Node-Exporter Server 
![image](https://github.com/CodeOps-Hub/Ansible/assets/156056444/15754c11-1a16-4615-b5c6-50dad330a968)

***

### Log Metrics of Salary-Node-Exporter Server
![image](https://github.com/CodeOps-Hub/Ansible/assets/156056444/c365d2f2-b37e-4d63-a3d0-9ae406ab23ac)

***

### Prometheus UI
![image](https://github.com/CodeOps-Hub/Ansible/assets/156056444/38aaa2e9-1b47-49d3-b68a-0e6fd9b5c022)

***

# Conclusion

This guide illustrates the process of deploying **Salary-node-exporter** through Ansible. By adhering to these instructions, you can effectively provision and set up `node exporter in Salary server` within your AWS infrastructure.

***
## Contact Information

|     Name         | Email  |
| -----------------| ------------------------------------ |
| Harshit Singh    | harshit.singh.snaatak@mygurukulam.co |
***

## References

| Description                                   | References  |
| --------------------------------------------  | -------------------------------------------------|
|  Ansible Doc | [Reference link](https://docs.ansible.com/ansible/latest/index.html) |
| Node Exporter Setup | [Reference link](https://codewizardly.com/prometheus-on-aws-ec2-part2/) |
| Reference Link For Ansible Dynamic Inventory | [Reference link](https://devopscube.com/setup-ansible-aws-dynamic-inventory/) |
| Node Exporter Releases| [Reference link](https://github.com/prometheus/node_exporter/releases) |
