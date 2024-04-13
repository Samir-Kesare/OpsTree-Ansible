
# Ansible Role to setup Elasticsearch

![image](https://github.com/CodeOps-Hub/Ansible/assets/156056746/bd5d87db-0ab5-46ae-ae06-bb6931b2a3af)

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
+ [Open Elasticsearch](#post-installation-steps)
+ [Conclusion](#conclusion)
+ [Contact Information](#contact-information)
+ [References](#references)

***
## Introduction
Elasticsearch is a distributed, RESTful search and analytics engine capable of solving a growing number of use cases. It provides real-time search and analytics capabilities and is widely used for log and event data analysis, full-text search, and more.

This documentation outlines the steps to set up Elasticsearch using Ansible, a powerful automation tool. Ansible allows for the automation of various tasks, including software provisioning, configuration management, and application deployment.


***
## Flow Diagram

![image](https://github.com/CodeOps-Hub/Ansible/assets/156056746/7db0d7ac-34fa-4597-b8de-0bcc41e9753d)


***
## Pre-requisites

Before using this Ansible role to set up Kibana, ensure that the following prerequisites are met:

1. **Ansible:**
   - Ansible must be installed on the control machine from which you plan to run the playbook. If Ansible is not installed, you can install it using this [link](https://github.com/CodeOps-Hub/Ansible/tree/Shikha/ElasticSearch-Ansible-Role/Dynamic-Inventory/roles/elasticsearc/es_installation) . Version used for POC : ansible 2.10.8


### Ansible 
![ansible_logo_icon_169596](https://github.com/avengers-p7/Documentation/assets/156056344/21281851-6cfa-4b18-aee4-8812e193dc62)

Ansible is an open-source automation tool that simplifies and accelerates IT infrastructure management, application deployment, and task automation. Employing a declarative language, Ansible enables users to define desired states for systems and applications, automating complex workflows efficiently. With agentless architecture, it connects to remote systems over SSH or other protocols, making it versatile and easy to implement. 


2. **SSH Access to Target Servers:**
   - Ensure that you have SSH access to the target servers where Elasticsearch will be installed.

3. **Elasticsearch**
   - Ensure that the correct version of Elasticsearch is installed on running on the server.
   
***
# Steps 
* Before going further check  [*Ansible Role For Elasticsearch Installation*](https://github.com/CodeOps-Hub/Ansible/tree/Shikha/ElasticSearch-Ansible-Role/Dynamic-Inventory/roles/elasticsearc/es_installation)
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
    - elasticsearch

```

1. `plugin: aws_ec2`: Specifies the use of the aws_ec2 plugin as the dynamic inventory source. This plugin is designed to fetch information about EC2 instances in AWS.
2. `regions: - us-east-1`: Indicates the AWS region(s) from which the dynamic inventory should fetch information.
3. `elasticsearch: "'elasticsearch' in tags.Type"`: Creates an Ansible group named elasticsearch. This group includes EC2 instances where the tag named Type has a value of 'elasticsearch'. You can tag all your Elasticsearch instances accordingly.
***
**Step 3: Create Ansible Role**
* Create a new Ansible role which should follow this directory structure:
  
![Screenshot from 2024-04-07 22-51-41](https://github.com/CodeOps-Hub/Ansible/assets/156056746/87561421-e1f9-4c5f-901c-9fa3504dcb1f)
***

**Step 4: Elasticsearch_playbook.yml**
* This file is defining a set of tasks to be executed on hosts belonging to the ubuntu group.

```yaml
---
- hosts: aws_ec2
  remote_user: ubuntu
  become: true
  roles:
    - elasticsearch_installation

```
**Step 5: Tasks**
1. `main.yml`: This main.yml file is acting as an orchestrator, importing tasks from other task files. This separation of tasks into different files is a good practice for better organization, especially when dealing with complex configurations or roles.

```yaml
---
- name: Install prerequisites
  apt:
    name: "{{ prerequisites }}"
    state: present

- name: Add Elastic GPG key
  apt_key:
    url: "{{ key_url }}"
    state: present

- name: Add Elastic APT repository for Elasticsearch 
  apt_repository:
    repo: "{{ elasticsearch_repository }}"
    state: present

- name: Install Elasticsearch
  apt:
    name: elasticsearch={{ elasticsearch_version }}
    state: present

- name: Reload systemd daemon
  shell: systemctl daemon-reload

- name: Enable and start Elasticsearch service
  systemd:
    name: elasticsearch
    enabled: yes
    state: started

- name: Copy Elasticsearch configuration template
  template:
    src: elasticsearch.yml.j2
    dest: "{{ elasticsearch_config_path }}"
    owner: root
    group: root
    mode: '0644'
  notify: Restart Elasticsearch

- name: Reload systemd to apply changes
  service:
    name: elasticsearch
    state: started
  become: true

```

***
  2. variables: This role comes with  values for several variables that have been used in the role. You can find these variables in the `variables/main.yml` file within the role directory.

```yaml
# vars file for elasticsearch_installation.yml

prerequisites:
 - apt-transport-https
 - software-properties-common
key_url: "https://artifacts.elastic.co/GPG-KEY-elasticsearch"
elasticsearch_repository: "deb https://artifacts.elastic.co/packages/8.x/apt stable main" 
elasticsearch_config_path: "/etc/elasticsearch/elasticsearch.yml"
elasticsearch_version: "8.13.1"
elasticsearch_host: "0.0.0.0"
elasticsearch_port: "9200"

```
***
**Step 6: Templates for Configuration**
We need to create jinja2 template :
* To configure Elasticsearch

1. `Elasticsearch.yml.j2` template includes parameteters to configure Elasticsearch

```yaml
# For more configuration options see the configuration guide for Elasticsearch in
# https://www.elastic.co/guide/index.html
# ======================== Elasticsearch Configuration =========================
#
# NOTE: Elasticsearch comes with reasonable defaults for most settings.
#       Before you set out to tweak and tune the configuration, make sure you
#       understand what are you trying to accomplish and the consequences.
#
# The primary way of configuring a node is via this file. This template lists
# the most important settings you may want to configure for a production cluster.
#
# Please consult the documentation for further information on configuration options:
# https://www.elastic.co/guide/en/elasticsearch/reference/index.html
#
# ---------------------------------- Cluster -----------------------------------
#
# Use a descriptive name for your cluster:
#
#cluster.name: my-application
#
# ------------------------------------ Node ------------------------------------
#
# Use a descriptive name for the node:
#
#node.name: node-1
#
# Add custom attributes to the node:
#
#node.attr.rack: r1
#
# ----------------------------------- Paths ------------------------------------
#
# Path to directory where to store the data (separate multiple locations by comma):
#
path.data: /var/lib/elasticsearch
#
# Path to log files:
#
path.logs: /var/log/elasticsearch
#
# ----------------------------------- Memory -----------------------------------
#
# Lock the memory on startup:
#
#bootstrap.memory_lock: true
#
# Make sure that the heap size is set to about half the memory available
# on the system and that the owner of the process is allowed to use this
# limit.
#
# Elasticsearch performs poorly when the system is swapping the memory.
#
# ---------------------------------- Network -----------------------------------
#
# By default Elasticsearch is only accessible on localhost. Set a different
# address here to expose this node on the network:
#
network.host: 0.0.0.0
#
# By default Elasticsearch listens for HTTP traffic on the first free port it
# finds starting at 9200. Set a specific HTTP port here:
#
http.port: 9200
#
# For more information, consult the network module documentation.
#
# --------------------------------- Discovery ----------------------------------
#
# Pass an initial list of hosts to perform discovery when this node is started:
# The default list of hosts is ["127.0.0.1", "[::1]"]
#
#discovery.seed_hosts: ["host1", "host2"]
#
# Bootstrap the cluster using an initial set of master-eligible nodes:
#
#cluster.initial_master_nodes: ["node-1", "node-2"]
#
# For more information, consult the discovery and cluster formation module documentation.
#
# ---------------------------------- Various -----------------------------------
#
# Allow wildcard deletion of indices:
#
#action.destructive_requires_name: false

#----------------------- BEGIN SECURITY AUTO CONFIGURATION -----------------------
#
# The following settings, TLS certificates, and keys have been automatically      
# generated to configure Elasticsearch security features on 11-04-2024 07:29:36
#
# --------------------------------------------------------------------------------

# Enable security features
xpack.security.enabled: false

xpack.security.enrollment.enabled:  false

# Enable encryption for HTTP API client connections, such as Kibana, Logstash, and Agents
xpack.security.http.ssl:
  enabled: true
  keystore.path: certs/http.p12

# Enable encryption and mutual authentication between cluster nodes
xpack.security.transport.ssl:
  enabled: true
  verification_mode: certificate
  keystore.path: certs/transport.p12
  truststore.path: certs/transport.p12
# Create a new cluster with the current node only
# Additional nodes can still join the cluster later
cluster.initial_master_nodes: ["ip-3.84.50.141"]

# Allow HTTP API connections from anywhere
# Connections are encrypted and require user authentication
http.host: 0.0.0.0

# Allow other nodes to join the cluster from anywhere
# Connections are encrypted and mutually authenticated
#transport.host: 0.0.0.0

#----------------------- END SECURITY AUTO CONFIGURATION -------------------------


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

![Screenshot from 2024-04-07 22-04-29](https://github.com/CodeOps-Hub/Ansible/assets/156056746/ec28c32f-dfde-4b6c-b4a1-9f65ad621084)


***

## Post-Installation Setup
* Access Elasticsearch on `https://URL:9200`
  
![Screenshot from 2024-04-07 22-07-36](https://github.com/CodeOps-Hub/Ansible/assets/156056746/152f8b87-6ba5-4d83-849b-305e5dffde63)


***
## Conclusion 

This documentation provided a step-by-step guide to setting up Elasticsearch using Ansible. By automating the installation process, you can ensure consistency and efficiency in deploying Elasticsearch across your infrastructure.

***
## Contact Information

|Shikha Tripathi                 | shikha.tripathi.snaatak@mygurukulam.co                                                                                      
|---------------------------------|------------------------------------------------------------|

***
## References

| Title                                      | URL                                           |
|--------------------------------------------|-----------------------------------------------|
| Ansible documentation           | https://docs.ansible.com/ansible/latest/index.html    |
| Elasticsearch Installation          |https://www.elastic.co/guide/en/elasticsearch/reference/current/install-elasticsearch.html  |
| Dyanmic Inventory               | https://www.youtube.com/watch?v=junPdh2yvbU&t=454s | 









