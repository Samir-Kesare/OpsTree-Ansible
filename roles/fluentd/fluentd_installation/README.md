

# Ansible Role to setup Elasticsearch
![image](https://github.com/CodeOps-Hub/Ansible/assets/156056746/7213d3ba-522d-448d-87f8-5966a53511dc)



|   Authors        |  Created on   |  Version   | Last updated by | Last edited on |
| -----------------| --------------| -----------|---------------- | -------------- |
| Shikha Tripathi      |  06 April 2024   |     v1     | Shikha  Tripathi     | 07 April 2024    |

***
## Table of Contents
+ [Introduction](#Introduction)
+ [Flow Diagram](#flow-diagram)
+ [Pre-requisites](#pre-requisites)
+ [Setup Ansible Role](#steps)
+ [Output Verification](#output)
+ [Open Fluentd](#post-installation-steps)
+ [Conclusion](#conclusion)
+ [Contact Information](#contact-information)
+ [References](#references)

***
## Introduction
Introduction:
Fluentd is a popular open-source data collector that enables easy log collection, processing, and forwarding. This comprehensive guide provides detailed instructions on setting up Fluentd to ingest logs into Elasticsearch using Ansible, an automation tool.

***
## Flow Diagram
![image](https://github.com/CodeOps-Hub/Ansible/assets/156056746/5a7c82bb-f087-4c9d-8b0a-16951543829b)



***
## Pre-requisites

Before using this Ansible role to set up Kibana, ensure that the following prerequisites are met:

1. **Ansible:**
   - Ansible must be installed on the control machine from which you plan to run the playbook. If Ansible is not installed, you can install it using this [link]https://github.com/CodeOps-Hub/Ansible/tree/shikha/fluentd/redis/roles/fluentd/fluentd_installation) . Version used for POC : ansible 2.10.8


### Ansible 
![ansible_logo_icon_169596](https://github.com/avengers-p7/Documentation/assets/156056344/21281851-6cfa-4b18-aee4-8812e193dc62)

Ansible is an open-source automation tool that simplifies and accelerates IT infrastructure management, application deployment, and task automation. Employing a declarative language, Ansible enables users to define desired states for systems and applications, automating complex workflows efficiently. With agentless architecture, it connects to remote systems over SSH or other protocols, making it versatile and easy to implement. 


2. **SSH Access to Target Servers:**
   - Ensure that you have SSH access to the target servers where Elasticsearch will be installed.

3. **Elasticsearch**
   - Ensure that the correct version of Elasticsearch is installed on running on the server.
   
***
# Steps 
* Before going further check  [*Ansible Role For Elasticsearch Installation*](https://github.com/CodeOps-Hub/Ansible/tree/shikha/fluentd/redis/roles/fluentd/fluentd_installation)
* For more information on [Ansible Roles](https://github.com/avengers-p7/Documentation/blob/main/Application_CI/Design/DevOps%20Practices/Ansible/Ansible%20Role.md)

**Step 1: Dynamic Inventory Setup** 

```yaml
[defaults]

# some basic default values...

[defaults]
inventory = aws_ec2.yml
host_key_checking = False
remote_user = ubuntu
private_key_file = /home/ubuntu/ansible.pem
[inventory]
enable_plugins = aws_ec2

```

> [!NOTE]
>Ensure that for dynamic inventory you have the necessary AWS credentials configured in AWS CLI or an IAM role on the node. 

**Step 2:  AWS EC2 Inventory**

```yaml
---
plugin: aws_ec2
regions:
  - us-east-1
hostnames:
  - ip-address
filters:
  tag:Name:
    - fluentd

```

1. `plugin: aws_ec2`: Specifies the use of the aws_ec2 plugin as the dynamic inventory source. This plugin is designed to fetch information about EC2 instances in AWS.
2. `regions: - us-east-1`: Indicates the AWS region(s) from which the dynamic inventory should fetch information.
3. `elasticsearch: "'elasticsearch' in tags.Type"`: Creates an Ansible group named elasticsearch. This group includes EC2 instances where the tag named Type has a value of 'elasticsearch'. You can tag all your Elasticsearch instances accordingly.
***
**Step 3: Create Ansible Role**
* Create a new Ansible role which should follow this directory structure:
  
![image](https://github.com/CodeOps-Hub/Ansible/assets/156056746/8f07ff2b-7d37-4118-a561-93d976ffbde6)

***

**Step 4: Fluentd_playbook.yml**
* This file is defining a set of tasks to be executed on hosts belonging to the ubuntu group.

```yaml
------
- hosts: aws_ec2
  remote_user: ubuntu
  become: true
  roles:
    - fluentd

```
**Step 5: Tasks**
1. `main.yml`: This main.yml file is acting as an orchestrator, importing tasks from other task files. This separation of tasks into different files is a good practice for better organization, especially when dealing with complex configurations or roles.

```yaml
---
# tasks file for fluentd
- name: Install Fluentd
  become: yes
  apt:
    name: apt-transport-https
    state: present

- name: Add Treasure Data repository key
  become: yes
  shell: curl -L https://toolbelt.treasuredata.com/sh/install-ubuntu-bionic-td-agent3.sh | sh

- name: Install td-agent
  become: yes
  apt:
    name: td-agent
    state: present

- name: Install libcurl4-gnutls-dev
  become: yes
  apt:
    name: libcurl4-gnutls-dev
    state: present

- name: Install build-essential
  become: yes
  apt:
    name: build-essential
    state: present

- name: Install Ruby
  become: yes
  apt:
    name: ruby
    state: present

- name: Install Rubygems
  become: yes
  apt:
    name: rubygems
    state: present

- name: Install Ruby development headers
  become: yes
  apt:
    name: ruby-dev
    state: present
- name: Install fluent-plugin-elasticsearch
  become: yes
  gem:
    name: fluent-plugin-elasticsearch

- name: Copy Fluentd configuration file
  become: yes
  template:
    src: td-agent.conf.j2
    dest: /etc/td-agent/td-agent.conf
    owner: td-agent
    group: td-agent
    mode: '0640'
  notify: restart td-agent service

- name: Ensure /var/log/td-agent directory exists
  become: yes
  file:
    path: /var/log/td-agent
    state: directory
```


**Step 6: Templates for Configuration**
We need to create jinja2 template :
* To configure Fluentd

1. `td-agent.conf.j2` template includes parameteters to configure Elasticsearch

```yaml

# td-agent.conf
<source>
  @type tail
  path /var/log/redis/redis-server.log
  pos_file /var/log/td-agent/syslog.log.pos
  tag prod-backend-system-logs
  <parse>
  @type regexp
  expression /(?<message>[\s\S]*)/
  </parse>
</source>

<match prod-backend-system-logs>
  @type elasticsearch
  host 3.87.248.148:9200
  port 9200
  logstash_format true
  logstash_prefix prod-backend-system-logs
  enable_ilm true
  flush_interval 10s
</match>
```

**Step 7: Playbook Execution**

* To set up Elasticsearch on your target servers, you will execute the Ansible playbook using the following command:

```bash
ansible-playbook -i aws_ec2.yml playbook.yml
```

> Additional Options
> 
> --limit: You can use this option to specify a subset of hosts from the inventory on which the playbook should be executed.
> 
> -e or --extra-vars: You can pass extra variables to the playbook using this option.


***
## Output
**Host-level output**: Output for each host would indicate whether the playbook execution was successful or not.

![Screenshot from 2024-04-08 12-53-24](https://github.com/CodeOps-Hub/Ansible/assets/156056746/1dd1c1fe-8c91-4529-95e9-90bbb1b6fa33)


***

## Post-Installation Setup
* Access Elasticsearch on `https://URL:9200`
  
![Screenshot from 2024-04-08 13-00-44](https://github.com/CodeOps-Hub/Ansible/assets/156056746/43032d61-4b09-49af-9c07-5d78fe64e15d)


***
## Conclusion 
Fluentd, along with Ansible automation, simplifies log management and analysis by efficiently collecting and forwarding logs to Elasticsearch. This setup enhances system monitoring and troubleshooting capabilities.

***
## Contact Information

|Shikha Tripathi                 | shikha.tripathi.snaatak@mygurukulam.co                                                                                      
|---------------------------------|------------------------------------------------------------|

***
## References

| Title                                      | URL                                           |
|--------------------------------------------|-----------------------------------------------|
| Ansible documentation           | https://docs.ansible.com/ansible/latest/index.html    |
| Fluentd Installation          | https://docs.fluentd.org/installation/install-by-deb |
| Dyanmic Inventory               | https://www.youtube.com/watch?v=junPdh2yvbU&t=454s | 









