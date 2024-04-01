# Postgres-Node-Exporter Ansible Role Documentation

<img width="513" alt="image" src="https://github.com/CodeOps-Hub/Ansible/assets/156057205/12cd5b68-8b90-473d-8a40-30830083733d">

***

| **Author** | **Created on** | **Last Updated** | **Document Version** |
| ---------- | -------------- | ---------------- | -------------------- |
| **Shreya Jaiswal** | **02 April 2024** | **02 April 2024** | **v1** |

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

This role is designed to automate the installation and configuration of Node Exporter on target ubuntu server(Postgres). This role aims to simplify the process of setting up Postgres Node Exporter server.

***

# Flow Diagram

<img width="900" alt="image" src="https://github.com/CodeOps-Hub/Ansible/assets/156057205/347e6f11-4081-4242-8647-be2966452687">

***

# Pre-requisites

| **Pre-requisite** | **Description** |
| ----------------- | --------------- |
| **Ansible**       | Ansible must be installed on the control machine from which you plan to run the playbook. |
| **AWS CLI**       | AWS CLI for providing aws credentials to fetch resources. |
| **Python**        | Ensure that Python is installed on your system. Ansible relies on Python for its execution, and dynamic inventory scripts are typically written in Python. |
| **PIP (Python Package Installer)** | Install pip if it's not already installed. pip is a package manager for Python that allows you to install and manage Python packages. |
| **Boto3**   |  If your dynamic inventory script relies on AWS APIs to fetch inventory data, you'll need to install `boto3` using `pip`. |
| **Postgres Server** | Must have installed `postgres` with `security group` having necessary ports allowed on it (22, 9090 for prometheus, 9100 for node exporter, 5432 for postgresql). |
| **Prometheus Server** | Updated Prometheus Server for moniotoring logs generated through `postgres-node-exporter` with `security group` having necessary ports allowed on it (22, 9090 for prometheus, 9100 for node exporter). |

***

# Directory Structure

<img width="400" alt="image" src="https://github.com/CodeOps-Hub/Ansible/assets/156057205/aa2b6dad-6eec-4cae-aaa3-af27946d5180">

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
roles_path=Node_Exporter_role
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
    - Postgres

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
    - Node_Exporter_role

```
</details>

***

## Postgres-Node-Exporter Role

### handlers file (main.yml)

This Ansible task restarts the Node Exporter service named "postgres_node_exporter" using the systemd module. It ensures that the service is in the "restarted" state, effectively restarting the Node Exporter service on the target system.

<details>
<summary> Click here to see main.yml file</summary>
<br>
  
```shell
---
- name: Restart Node Exporter
  systemd:
    name: postgres_node_exporter
    state: restarted

```
</details>

***

### meta file (main.yml)

This is a metadata file (meta/main.yml) for an Ansible role, providing essential information about the role such as author, description, company, license, and supported platforms.

<details>
<summary> Click here to see main.yml file</summary>
<br>
  
```shell
galaxy_info:
  author: Shreya
  description: Snaatak-P7
  company: Mygurukulam- Powered by Opstree

  # If the issue tracker for your role is not on github, uncomment the
  # next line and provide a value
  # issue_tracker_url: http://example.com/issue/tracker

  # Choose a valid license ID from https://spdx.org - some suggested licenses:
  # - BSD-3-Clause (default)
  # - MIT
  # - GPL-2.0-or-later
  # - GPL-3.0-only
  # - Apache-2.0
  # - CC-BY-4.0
  license: license (GPL-2.0-or-later, MIT, etc)

  min_ansible_version: 2.1

  # If this a Container Enabled role, provide the minimum Ansible Container version.
  # min_ansible_container_version:

  #
  # Provide a list of supported platforms, and for each platform a list of versions.
  # If you don't wish to enumerate all versions for a particular platform, use 'all'.
  # To view available platforms and versions (or releases), visit:
  # https://galaxy.ansible.com/api/v1/platforms/
  #
  # platforms:
  # - name: Fedora
  #   versions:
  #   - all
  #   - 25
  # - name: SomePlatform
  #   versions:
  #   - all
  #   - 1.0
  #   - 7
  #   - 99.99

  galaxy_tags: []
    # List tags for your role here, one per line. A tag is a keyword that describes
    # and categorizes the role. Users find roles by searching for tags. Be sure to
    # remove the '[]' above, if you add tags to this list.
    #
    # NOTE: A tag is limited to a single word comprised of alphanumeric characters.
    #       Maximum 20 tags per role.

dependencies: []
  # List your role dependencies here, one per line. Be sure to remove the '[]' above,
  # if you add dependencies to this list.

```
</details>

***

### tasks file (main.yml)

This Ansible playbook consists of tasks aimed at downloading, extracting, installing, and starting the Node Exporter service.

<details>
<summary> Click here to see main.yml file</summary>
<br>
  
```shell
---
- name: Download Node Exporter binary
  get_url:
    url: "{{url}}"
    dest: "{{dest}}"

- name: Ensure "{{node_exporter_install_dir}}" directory exists
  file:
    path: "{{node_exporter_install_dir}}"
    state: directory

- name: Extract Node Exporter archive
  ansible.builtin.unarchive:
    src: "{{dest}}"
    dest: "{{ node_exporter_install_dir }}"
    remote_src: yes

- name: Install Node Exporter service
  ansible.builtin.template:
    src: "{{src}}"
    dest: "{{src_dest}}"
  notify: Restart Node Exporter

- name: Start Node Exporter service
  ansible.builtin.service:
    name: postgres_node_exporter
    state: started

```
</details>

***

### templates file (postgres_node_exporter.service.j2)

This configuration sets up a systemd service for Prometheus Node Exporter tailored for PostgreSQL monitoring. It ensures that the service starts after the network is available. The service runs as the user and group "node_exporter" and launches Node Exporter with PostgreSQL-specific collectors enabled, listening on the specified address and port. Finally, it specifies that the service should be enabled and started during the multi-user boot sequence.

<details>
<summary> Click here to see prometheus_config.j2 file</summary>
<br>
  
```shell
[Unit]
Description=Prometheus Node Exporter for PostgreSQL
After=network.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart={{ node_exporter_install_dir }}/node_exporter --web.listen-address={{ node_exporter_listen_address }} --web.telemetry-path={{ node_exporter_telemetry_path }} --collector.postgres --collector.postgres-address={{ postgres_host }}:{{ postgres_port }}

[Install]
WantedBy=multi-user.target

```
</details>

***

### tests file (test.yml)

This Ansible playbook named "Install node exporter" targets the localhost machine and utilizes the "Node_Exporter_role" role to install Node Exporter in Postgres Server.

<details>
<summary> Click here to see test.yml file</summary>
<br>
  
```shell
---
- name: Install node exporter
  hosts: localhost
  become: yes
  roles:
    - Node_Exporter_role

```
</details>

***

### vars file (main.yml)

This YAML configuration sets parameters for the Node Exporter and PostgreSQL monitoring. It specifies the Node Exporter version, installation directory, listening address, telemetry path, and URL for downloading the Node Exporter binary.

<details>
<summary> Click here to see main.yml file</summary>
<br>
  
```shell
---
node_exporter_version: "1.7.0"
node_exporter_install_dir: "/opt/node_exporter"
node_exporter_listen_address: "0.0.0.0:9100"
node_exporter_telemetry_path: "/metrics"
postgres_host: "localhost"
postgres_port: "5432"
url: "https://github.com/prometheus/node_exporter/releases/download/v{{ node_exporter_version }}/node_exporter-{{ node_exporter_version }}.linux-amd64.tar.gz"
dest: "/tmp/node_exporter.tar.gz"
src: "postgres_node_exporter.service.j2"
src_dest: "/etc/systemd/system/postgres_node_exporter.service"

```
</details>

***

# Output

### Node Exporter Role Execution

<img width="900" alt="image" src="https://github.com/CodeOps-Hub/Ansible/assets/156057205/d799d979-6783-4324-b4dd-eda92e443f17">

***

### Postgres-Node-Exporter Server 

<img width="900" alt="image" src="https://github.com/CodeOps-Hub/Ansible/assets/156057205/7a735002-e3cc-44d6-ae57-eabade14d420">

***
<img width="900" alt="image" src="https://github.com/CodeOps-Hub/Ansible/assets/156057205/442a0d16-b252-4f9d-98e3-391d969a8eab">

***

### Logs in Prometheus Server

<img width="900" alt="image" src="https://github.com/CodeOps-Hub/Ansible/assets/156057205/50b8dd35-4c05-494f-965f-fa6b2e3c2c6e">

***

### Log Metrics of Postgres-Node-Exporter Server

<img width="900" alt="image" src="https://github.com/CodeOps-Hub/Ansible/assets/156057205/141fd038-8706-448a-9fc3-f4a840472d7a">

***

# Conclusion

This guide illustrates the process of deploying **postgres-node-exporter** through Ansible. By adhering to these instructions, you can effectively provision and set up `node exporter in postgres server` within your AWS infrastructure.

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
| [Link](https://https://stackoverflow.com/questions/72449595/ansible-role-for-node-exporter-installation) | Node Exporter Setup. |
| [Link](https://www.youtube.com/watch?v=junPdh2yvbU&t=454s) | Reference Link For Ansible Dynamic Inventory. |
