# Ansible role for Attendance Node Exporter

|   Author        |  Created on   |  Version   | Last updated by  | Last edited on |
| --------------- | --------------| -----------|----------------- | -------------- |
| Khushi Malhotra |  02 April 2024  |  Version 1 | Khushi Malhotra  | 03 April 2024    |

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
