
# Ansible Role to setup Kibana

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
   - Ensure that you have SSH access to the target servers where Kibana will be installed.

3. **Elasticsearch**
   - Ensure that the correct version of Elasticsearch is installed on running on the server.
   
***

## Kibana 
Kibana is an open-source data visualization and exploration tool designed primarily for analyzing and visualizing log and time-series data stored in Elasticsearch. It is part of the Elastic Stack (formerly known as the ELK Stack), which also includes Elasticsearch, Logstash, and Beats.

Kibana provides a user-friendly interface for performing various tasks such as:
- Data Visualization: It allows users to create a wide range of visualizations including line charts, bar charts, pie charts, maps, and more to represent data stored in Elasticsearch.
- Dashboard Creation: Users can combine multiple visualizations into interactive dashboards, enabling them to gain insights from complex data sets at a glance.
- Data Exploration: Kibana provides powerful search capabilities and filtering options, allowing users to explore their data efficiently.
- Monitoring and Alerting: Kibana can be used to monitor the health and performance of Elasticsearch clusters, as well as set up alerts based on predefined conditions.

Overall, Kibana plays a crucial role in the Elastic Stack ecosystem by providing users with the tools they need to analyze and visualize their data effectively.

Please refer [*Kibana Documentation*](https://www.elastic.co/guide/en/kibana/current/index.html) for better understanding of Kibana

# Steps 
* Before going further check  [*Ansible Role For Kibana Installation*](https://github.com/CodeOps-Hub/ansible/tree/main/roles/kibana)
* For more information on [Ansible Roles](https://github.com/avengers-p7/Documentation/blob/main/Application_CI/Design/DevOps%20Practices/Ansible/Ansible%20Role.md)

**Step 1: Dynamic Inventory Setup** 

```yaml
[defaults]

# some basic default values...


# Use AWS EC2 dynamic inventory for managing hosts
inventory      = aws_ec2.yml

# Disable SSH host key checking for convenience.
host_key_checking = False

# Specify the path to the private key file for SSH connections.
private_key_file = privateKey.pem

Sets the remote user for SSH connections to 'ubuntu'
remote_user = ubuntu

[inventory]
# enable inventory plugins, default: 'host_list', 'script', 'auto', 'yaml', 'ini', 'toml'
enable_plugins = aws_ec2, host_list, virtualbox, yaml, constructed, script, auto, ini, toml
```

> [!NOTE]
>Ensure that for dynamic inventory you have the necessary AWS credentials configured in AWS CLI or an IAM role on the node. 

**Step 2:  AWS EC2 Inventory**

```yaml
---
plugin: aws_ec2
regions:
  - us-east-1

groups: 
  efk: "'efk' in tags.Type"
```

1. `plugin: aws_ec2`: Specifies the use of the aws_ec2 plugin as the dynamic inventory source. This plugin is designed to fetch information about EC2 instances in AWS.
2. `regions: - us-east-1`: Indicates the AWS region(s) from which the dynamic inventory should fetch information.
3. `efk: "'efk' in tags.Type"`: Creates an Ansible group named efk. This group includes EC2 instances where the tag named Type has a value of 'efk'. You can tag all your EFK instances accordingly.

**Step 3: Create Ansible Role**
* Create a new Ansible role which should follow this directory structure:

![Screenshot 2024-04-07 164759](https://github.com/CodeOps-Hub/Documentation/assets/156056344/f0311895-9da9-4f8c-babc-59341e597d8c)




**Step 4: kibana_playbook.yml**
* This file is defining a set of tasks to be executed on hosts belonging to the ubuntu group.

```yaml
---
- hosts: efk
  become: yes
  gather_facts: yes 
  roles:
    - kibana
```
**Step 5: Tasks**
1. `main.yml`: This main.yml file is acting as an orchestrator, importing tasks from other task files. This separation of tasks into different files is a good practice for better organization, especially when dealing with complex configurations or roles.

```yaml
---
---
# tasks file for kibana
---

    - name: Install prerequisites
      apt:
        name: "{{ prerequisites }}"
        state: present

    - name: Add Elastic GPG key
      apt_key:
        url: "{{ key_url }}"
        state: present

    - name: Add Elastic APT repository for Kibana
      apt_repository:
        repo: "{{ kibana_repository }}"
        state: present

    - name: Install Kibana
      apt:
        name: kibana={{ kibana_version }}
        state: present

    - name: Reload systemd daemon
      shell: systemctl daemon-reload

    - name: Enable and start Kibana service
      systemd:
        name: kibana
        enabled: yes
        state: started

    - name: Copy Kibana configuration template
      template:
        src: kibana.yml.j2
        dest: "{{ kibana_config_path }}"
        owner: root
        group: root
        mode: '0644'
      notify: Restart Kibana


```

2. `Default` variables: This role comes with default values for several variables that have been used in the role. You can find these defaults in the `defaults/main.yml` file within the role directory.

```yaml
---
# defaults file for kibana
prerequisites:
 - apt-transport-https
 - software-properties-common
key_url: "https://artifacts.elastic.co/GPG-KEY-elasticsearch"
kibana_repository: "deb https://artifacts.elastic.co/packages/8.x/apt stable main"
kibana_config_path: "/etc/kibana/kibana.yml"
kibana_version: "8.13.1"
kibana_host: "0.0.0.0"
kibana_port: "5601"

```
### Important Role Variables 
| **Variable** | **Description** |
| ------------ | --------------- |
| kibana_version |  Version of Kibana to be installed |
| kibana_port | Default Kibana port |
| kibana_config_path | Kibana configuration file path |
| kibana_host | IP of the Elasticsearch/Kibana Host  |
| kibana_repository | Kibana repository |
| key_url | Kibana GPG Key URL |
| prerequisites | Pre-requisite packages for Kibana |

> [!NOTE]
> To customize the Kibana installation based on your specific requirements, you can override these default values in main.yaml file in the vars directory of the role. 



**Step 6: Templates for Configuration**
We need to create jinja2 template :
* To configure Kibana

1. `kibana.yml.j2` template includes parameteters to configure Kibana

```yaml
# For more configuration options see the configuration guide for Kibana in
# https://www.elastic.co/guide/index.html

# =================== System: Kibana Server ===================
# Kibana is served by a back end server. This setting specifies the port to use.
server.port: {{ kibana_port }}

# Specifies the address to which the Kibana server will bind. IP addresses and host names are both valid values.
# The default is 'localhost', which usually means remote machines will not be able to connect.
# To allow connections from remote users, set this parameter to a non-loopback address.
server.host: {{ kibana_host }}
# Enables you to specify a path to mount Kibana at if you are running behind a proxy.
# Use the `server.rewriteBasePath` setting to tell Kibana if it should remove the basePath
# from requests it receives, and to prevent a deprecation warning at startup.
# This setting cannot end in a slash.
#server.basePath: ""

# Specifies whether Kibana should rewrite requests that are prefixed with
# `server.basePath` or require that they are rewritten by your reverse proxy.
# Defaults to `false`.
#server.rewriteBasePath: false

# Specifies the public URL at which Kibana is available for end users. If
# `server.basePath` is configured this URL should end with the same basePath.
#server.publicBaseUrl: ""

# The maximum payload size in bytes for incoming server requests.
#server.maxPayload: 1048576

# The Kibana server's name. This is used for display purposes.
#server.name: "your-hostname"

# =================== System: Kibana Server (Optional) ===================
# Enables SSL and paths to the PEM-format SSL certificate and SSL key files, respectively.
# These settings enable SSL for outgoing requests from the Kibana server to the browser.
#server.ssl.enabled: false
#server.ssl.certificate: /path/to/your/server.crt
#server.ssl.key: /path/to/your/server.key

# =================== System: Elasticsearch ===================
# The URLs of the Elasticsearch instances to use for all your queries.
#elasticsearch.hosts: ["http://localhost:9200"]

# If your Elasticsearch is protected with basic authentication, these settings provide
# the username and password that the Kibana server uses to perform maintenance on the Kibana
# index at startup. Your Kibana users still need to authenticate with Elasticsearch, which
# is proxied through the Kibana server.
#elasticsearch.username: "kibana_system"
#elasticsearch.password: "pass"

# Kibana can also authenticate to Elasticsearch via "service account tokens".
# Service account tokens are Bearer style tokens that replace the traditional username/password based configuration.
# Use this token instead of a username/password.
# elasticsearch.serviceAccountToken: "my_token"

# Time in milliseconds to wait for Elasticsearch to respond to pings. Defaults to the value of
# the elasticsearch.requestTimeout setting.
#elasticsearch.pingTimeout: 1500

# Time in milliseconds to wait for responses from the back end or Elasticsearch. This value
# must be a positive integer.
#elasticsearch.requestTimeout: 30000

# The maximum number of sockets that can be used for communications with elasticsearch.
# Defaults to `Infinity`.
#elasticsearch.maxSockets: 1024

# Specifies whether Kibana should use compression for communications with elasticsearch
# Defaults to `false`.
#elasticsearch.compression: false

# List of Kibana client-side headers to send to Elasticsearch. To send *no* client-side
# headers, set this value to [] (an empty list).
#elasticsearch.requestHeadersWhitelist: [ authorization ]

# Header names and values that are sent to Elasticsearch. Any custom headers cannot be overwritten
# by client-side headers, regardless of the elasticsearch.requestHeadersWhitelist configuration.
#elasticsearch.customHeaders: {}

# Time in milliseconds for Elasticsearch to wait for responses from shards. Set to 0 to disable.
#elasticsearch.shardTimeout: 30000

# =================== System: Elasticsearch (Optional) ===================
# These files are used to verify the identity of Kibana to Elasticsearch and are required when
# xpack.security.http.ssl.client_authentication in Elasticsearch is set to required.
#elasticsearch.ssl.certificate: /path/to/your/client.crt
#elasticsearch.ssl.key: /path/to/your/client.key

# Enables you to specify a path to the PEM file for the certificate
# authority for your Elasticsearch instance.
#elasticsearch.ssl.certificateAuthorities: [ "/path/to/your/CA.pem" ]

# To disregard the validity of SSL certificates, change this setting's value to 'none'.
#elasticsearch.ssl.verificationMode: full

# =================== System: Logging ===================
# Set the value of this setting to off to suppress all logging output, or to debug to log everything. Defaults to 'info'
#logging.root.level: debug

# Enables you to specify a file where Kibana stores log output.
#logging.appenders.default:
#  type: file
#  fileName: /var/logs/kibana.log
#  layout:
#    type: json

# Example with size based log rotation
#logging.appenders.default:
#  type: rolling-file
#  fileName: /var/logs/kibana.log
#  policy:
#    type: size-limit
#    size: 256mb
#  strategy:
#    type: numeric
#    max: 10
#  layout:
#    type: json

# Logs queries sent to Elasticsearch.
#logging.loggers:
#  - name: elasticsearch.query
#    level: debug

# Logs http responses.
#logging.loggers:
#  - name: http.server.response
#    level: debug

# Logs system usage information.
#logging.loggers:
#  - name: metrics.ops
#    level: debug

# Enables debug logging on the browser (dev console)
#logging.browser.root:
#  level: debug

# =================== System: Other ===================
# The path where Kibana stores persistent data not saved in Elasticsearch. Defaults to data
#path.data: data

# Specifies the path where Kibana creates the process ID file.
#pid.file: /run/kibana/kibana.pid

# Set the interval in milliseconds to sample system and process performance
# metrics. Minimum is 100ms. Defaults to 5000ms.
#ops.interval: 5000

# Specifies locale to be used for all localizable strings, dates and number formats.
# Supported languages are the following: English (default) "en", Chinese "zh-CN", Japanese "ja-JP", French "fr-FR".
#i18n.locale: "en"

# =================== Frequently used (Optional)===================

# =================== Saved Objects: Migrations ===================
# Saved object migrations run at startup. If you run into migration-related issues, you might need to adjust these settings.

# The number of documents migrated at a time.
# If Kibana can't start up or upgrade due to an Elasticsearch `circuit_breaking_exception`,
# use a smaller batchSize value to reduce the memory pressure. Defaults to 1000 objects per batch.
#migrations.batchSize: 1000

# The maximum payload size for indexing batches of upgraded saved objects.
# To avoid migrations failing due to a 413 Request Entity Too Large response from Elasticsearch.
# This value should be lower than or equal to your Elasticsearch cluster’s `http.max_content_length`
# configuration option. Default: 100mb
#migrations.maxBatchSizeBytes: 100mb

# The number of times to retry temporary migration failures. Increase the setting
# if migrations fail frequently with a message such as `Unable to complete the [...] step after
# 15 attempts, terminating`. Defaults to 15
#migrations.retryAttempts: 15

# =================== Search Autocomplete ===================
# Time in milliseconds to wait for autocomplete suggestions from Elasticsearch.
# This value must be a whole number greater than zero. Defaults to 1000ms
#unifiedSearch.autocomplete.valueSuggestions.timeout: 1000

# Maximum number of documents loaded by each shard to generate autocomplete suggestions.
# This value must be a whole number greater than zero. Defaults to 100_000
#unifiedSearch.autocomplete.valueSuggestions.terminateAfter: 100000

```

**Step 7: Playbook Execution**

* To set up Kibanas on your target servers, you will execute the Ansible playbook using the following command:

```bash
ansible-playbook -i aws_ec2.yml kibana_playbook.yml
```

> Additional Options
> 
> --limit: You can use this option to specify a subset of hosts from the inventory on which the playbook should be executed.
> 
> -e or --extra-vars: You can pass extra variables to the playbook using this option.


***
## Output
**Host-level output**: Output for each host would indicate whether the playbook execution was successful or not.

![Screenshot 2024-04-07 155342](https://github.com/CodeOps-Hub/Documentation/assets/156056344/7d76ab9e-91d6-4af8-bd79-135ed5fdf850)


***

## Post-Installation Setup
* Access Kibana on `http://URL:5601`
  
![Screenshot 2024-04-07 155322](https://github.com/CodeOps-Hub/Documentation/assets/156056344/12509da0-2431-4499-8275-bac5a5f4f929)


***
## Conclusion 

* This guide illustrates the process of deploying Kibana through Ansible. By adhering to these instructions, you can effectively provision and set up redis within your AWS infrastructure.

***
## Contact Information

|Aakash Tripathi                 | aakash.tripathi.snaatak@mygurukulam.co                                                                                      
|---------------------------------|------------------------------------------------------------|

***
## References

| Title                                      | URL                                           |
|--------------------------------------------|-----------------------------------------------|
| Ansible documentation           | https://docs.ansible.com/ansible/latest/index.html    |
| Kibana Installation          | https://www.elastic.co/guide/en/kibana/current/deb.html |
| Kibana Default Configurations | https://github.com/elastic/kibana/blob/main/config/kibana.yml |
| Dyanmic Inventory               | https://www.youtube.com/watch?v=junPdh2yvbU&t=454s | 









# Ansible Role to setup Kibana

![images](https://github.com/CodeOps-Hub/Documentation/assets/156056344/27b67a6b-4461-4b33-b51e-c5af850beb28)



|   Authors        |  Created on   |  Version   | Last updated by | Last edited on |
| -----------------| --------------| -----------|---------------- | -------------- |
| Aakash Tripathi      |  04 April 2024   |     v1     | Aakash Tripathi     | 05 April 2024    |

***
## Table of Contents
+ [Introduction](#Introduction)
+ [Flow Diagram](#flow-diagram)
+ [Pre-requisites](#pre-requisites)
+ [Setup Ansible Role](#steps)
+ [Output Verification](#output)
+ [Open Kibana](#post-installation-steps)
+ [Conclusion](#conclusion)
+ [Contact Information](#contact-information)
+ [References](#references)

***
## Introduction
This role is designed to automate the installation and configuration of Kibana on target ubuntu servers. This role aims to simplify the process of setting up kibana visualizations on  servers running Elasticsearch.

***
## Flow Diagram

* This diagram should help you visualize the sequence of tasks in the Ansible role for setting up Kibana.
![Screenshot 2024-04-07 170428](https://github.com/CodeOps-Hub/Documentation/assets/156056344/4186264e-ec9e-4e14-9b15-f866f19eafb0)



***
## Pre-requisites

Before using this Ansible role to set up Kibana, ensure that the following prerequisites are met:

1. **Ansible:**
   - Ansible must be installed on the control machine from which you plan to run the playbook. If Ansible is not installed, you can install it using this [link](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) . Version used for POC : ansible 2.10.8


### Ansible 
![ansible_logo_icon_169596](https://github.com/avengers-p7/Documentation/assets/156056344/21281851-6cfa-4b18-aee4-8812e193dc62)

Ansible is an open-source automation tool that simplifies and accelerates IT infrastructure management, application deployment, and task automation. Employing a declarative language, Ansible enables users to define desired states for systems and applications, automating complex workflows efficiently. With agentless architecture, it connects to remote systems over SSH or other protocols, making it versatile and easy to implement. 


2. **SSH Access to Target Servers:**
   - Ensure that you have SSH access to the target servers where Kibana will be installed.

3. **Elasticsearch**
   - Ensure that the correct version of Elasticsearch is installed on running on the server.
   
***

## Kibana 
Kibana is an open-source data visualization and exploration tool designed primarily for analyzing and visualizing log and time-series data stored in Elasticsearch. It is part of the Elastic Stack (formerly known as the ELK Stack), which also includes Elasticsearch, Logstash, and Beats.

Kibana provides a user-friendly interface for performing various tasks such as:
- Data Visualization: It allows users to create a wide range of visualizations including line charts, bar charts, pie charts, maps, and more to represent data stored in Elasticsearch.
- Dashboard Creation: Users can combine multiple visualizations into interactive dashboards, enabling them to gain insights from complex data sets at a glance.
- Data Exploration: Kibana provides powerful search capabilities and filtering options, allowing users to explore their data efficiently.
- Monitoring and Alerting: Kibana can be used to monitor the health and performance of Elasticsearch clusters, as well as set up alerts based on predefined conditions.

Overall, Kibana plays a crucial role in the Elastic Stack ecosystem by providing users with the tools they need to analyze and visualize their data effectively.

Please refer [*Kibana Documentation*](https://www.elastic.co/guide/en/kibana/current/index.html) for better understanding of Kibana

# Steps 
* Before going further check  [*Ansible Role For Kibana Installation*](https://github.com/CodeOps-Hub/ansible/tree/main/roles/kibana)
* For more information on [Ansible Roles](https://github.com/avengers-p7/Documentation/blob/main/Application_CI/Design/DevOps%20Practices/Ansible/Ansible%20Role.md)

**Step 1: Dynamic Inventory Setup** 

```yaml
[defaults]

# some basic default values...


# Use AWS EC2 dynamic inventory for managing hosts
inventory      = aws_ec2.yml

# Disable SSH host key checking for convenience.
host_key_checking = False

# Specify the path to the private key file for SSH connections.
private_key_file = privateKey.pem

Sets the remote user for SSH connections to 'ubuntu'
remote_user = ubuntu

[inventory]
# enable inventory plugins, default: 'host_list', 'script', 'auto', 'yaml', 'ini', 'toml'
enable_plugins = aws_ec2, host_list, virtualbox, yaml, constructed, script, auto, ini, toml
```

> [!NOTE]
>Ensure that for dynamic inventory you have the necessary AWS credentials configured in AWS CLI or an IAM role on the node. 

**Step 2:  AWS EC2 Inventory**

```yaml
---
plugin: aws_ec2
regions:
  - us-east-1

groups: 
  efk: "'efk' in tags.Type"
```

1. `plugin: aws_ec2`: Specifies the use of the aws_ec2 plugin as the dynamic inventory source. This plugin is designed to fetch information about EC2 instances in AWS.
2. `regions: - us-east-1`: Indicates the AWS region(s) from which the dynamic inventory should fetch information.
3. `efk: "'efk' in tags.Type"`: Creates an Ansible group named efk. This group includes EC2 instances where the tag named Type has a value of 'efk'. You can tag all your EFK instances accordingly.

**Step 3: Create Ansible Role**
* Create a new Ansible role which should follow this directory structure:

![Screenshot 2024-04-07 164759](https://github.com/CodeOps-Hub/Documentation/assets/156056344/f0311895-9da9-4f8c-babc-59341e597d8c)




**Step 4: kibana_playbook.yml**
* This file is defining a set of tasks to be executed on hosts belonging to the ubuntu group.

```yaml
---
- hosts: efk
  become: yes
  gather_facts: yes 
  roles:
    - kibana
```
**Step 5: Tasks**
1. `main.yml`: This main.yml file is acting as an orchestrator, importing tasks from other task files. This separation of tasks into different files is a good practice for better organization, especially when dealing with complex configurations or roles.

```yaml
---
---
# tasks file for kibana
---

    - name: Install prerequisites
      apt:
        name: "{{ prerequisites }}"
        state: present

    - name: Add Elastic GPG key
      apt_key:
        url: "{{ key_url }}"
        state: present

    - name: Add Elastic APT repository for Kibana
      apt_repository:
        repo: "{{ kibana_repository }}"
        state: present

    - name: Install Kibana
      apt:
        name: kibana={{ kibana_version }}
        state: present

    - name: Reload systemd daemon
      shell: systemctl daemon-reload

    - name: Enable and start Kibana service
      systemd:
        name: kibana
        enabled: yes
        state: started

    - name: Copy Kibana configuration template
      template:
        src: kibana.yml.j2
        dest: "{{ kibana_config_path }}"
        owner: root
        group: root
        mode: '0644'
      notify: Restart Kibana


```

2. `Default` variables: This role comes with default values for several variables that have been used in the role. You can find these defaults in the `defaults/main.yml` file within the role directory.

```yaml
---
# defaults file for kibana
prerequisites:
 - apt-transport-https
 - software-properties-common
key_url: "https://artifacts.elastic.co/GPG-KEY-elasticsearch"
kibana_repository: "deb https://artifacts.elastic.co/packages/8.x/apt stable main"
kibana_config_path: "/etc/kibana/kibana.yml"
kibana_version: "8.13.1"
kibana_host: "0.0.0.0"
kibana_port: "5601"

```
### Important Role Variables 
| **Variable** | **Description** |
| ------------ | --------------- |
| kibana_version |  Version of Kibana to be installed |
| kibana_port | Default Kibana port |
| kibana_config_path | Kibana configuration file path |
| kibana_host | IP of the Elasticsearch/Kibana Host  |
| kibana_repository | Kibana repository |
| key_url | Kibana GPG Key URL |
| prerequisites | Pre-requisite packages for Kibana |

> [!NOTE]
> To customize the Kibana installation based on your specific requirements, you can override these default values in main.yaml file in the vars directory of the role. 



**Step 6: Templates for Configuration**
We need to create jinja2 template :
* To configure Kibana

1. `kibana.yml.j2` template includes parameteters to configure Kibana

```yaml
# For more configuration options see the configuration guide for Kibana in
# https://www.elastic.co/guide/index.html

# =================== System: Kibana Server ===================
# Kibana is served by a back end server. This setting specifies the port to use.
server.port: {{ kibana_port }}

# Specifies the address to which the Kibana server will bind. IP addresses and host names are both valid values.
# The default is 'localhost', which usually means remote machines will not be able to connect.
# To allow connections from remote users, set this parameter to a non-loopback address.
server.host: {{ kibana_host }}
# Enables you to specify a path to mount Kibana at if you are running behind a proxy.
# Use the `server.rewriteBasePath` setting to tell Kibana if it should remove the basePath
# from requests it receives, and to prevent a deprecation warning at startup.
# This setting cannot end in a slash.
#server.basePath: ""

# Specifies whether Kibana should rewrite requests that are prefixed with
# `server.basePath` or require that they are rewritten by your reverse proxy.
# Defaults to `false`.
#server.rewriteBasePath: false

# Specifies the public URL at which Kibana is available for end users. If
# `server.basePath` is configured this URL should end with the same basePath.
#server.publicBaseUrl: ""

# The maximum payload size in bytes for incoming server requests.
#server.maxPayload: 1048576

# The Kibana server's name. This is used for display purposes.
#server.name: "your-hostname"

# =================== System: Kibana Server (Optional) ===================
# Enables SSL and paths to the PEM-format SSL certificate and SSL key files, respectively.
# These settings enable SSL for outgoing requests from the Kibana server to the browser.
#server.ssl.enabled: false
#server.ssl.certificate: /path/to/your/server.crt
#server.ssl.key: /path/to/your/server.key

# =================== System: Elasticsearch ===================
# The URLs of the Elasticsearch instances to use for all your queries.
#elasticsearch.hosts: ["http://localhost:9200"]

# If your Elasticsearch is protected with basic authentication, these settings provide
# the username and password that the Kibana server uses to perform maintenance on the Kibana
# index at startup. Your Kibana users still need to authenticate with Elasticsearch, which
# is proxied through the Kibana server.
#elasticsearch.username: "kibana_system"
#elasticsearch.password: "pass"

# Kibana can also authenticate to Elasticsearch via "service account tokens".
# Service account tokens are Bearer style tokens that replace the traditional username/password based configuration.
# Use this token instead of a username/password.
# elasticsearch.serviceAccountToken: "my_token"

# Time in milliseconds to wait for Elasticsearch to respond to pings. Defaults to the value of
# the elasticsearch.requestTimeout setting.
#elasticsearch.pingTimeout: 1500

# Time in milliseconds to wait for responses from the back end or Elasticsearch. This value
# must be a positive integer.
#elasticsearch.requestTimeout: 30000

# The maximum number of sockets that can be used for communications with elasticsearch.
# Defaults to `Infinity`.
#elasticsearch.maxSockets: 1024

# Specifies whether Kibana should use compression for communications with elasticsearch
# Defaults to `false`.
#elasticsearch.compression: false

# List of Kibana client-side headers to send to Elasticsearch. To send *no* client-side
# headers, set this value to [] (an empty list).
#elasticsearch.requestHeadersWhitelist: [ authorization ]

# Header names and values that are sent to Elasticsearch. Any custom headers cannot be overwritten
# by client-side headers, regardless of the elasticsearch.requestHeadersWhitelist configuration.
#elasticsearch.customHeaders: {}

# Time in milliseconds for Elasticsearch to wait for responses from shards. Set to 0 to disable.
#elasticsearch.shardTimeout: 30000

# =================== System: Elasticsearch (Optional) ===================
# These files are used to verify the identity of Kibana to Elasticsearch and are required when
# xpack.security.http.ssl.client_authentication in Elasticsearch is set to required.
#elasticsearch.ssl.certificate: /path/to/your/client.crt
#elasticsearch.ssl.key: /path/to/your/client.key

# Enables you to specify a path to the PEM file for the certificate
# authority for your Elasticsearch instance.
#elasticsearch.ssl.certificateAuthorities: [ "/path/to/your/CA.pem" ]

# To disregard the validity of SSL certificates, change this setting's value to 'none'.
#elasticsearch.ssl.verificationMode: full

# =================== System: Logging ===================
# Set the value of this setting to off to suppress all logging output, or to debug to log everything. Defaults to 'info'
#logging.root.level: debug

# Enables you to specify a file where Kibana stores log output.
#logging.appenders.default:
#  type: file
#  fileName: /var/logs/kibana.log
#  layout:
#    type: json

# Example with size based log rotation
#logging.appenders.default:
#  type: rolling-file
#  fileName: /var/logs/kibana.log
#  policy:
#    type: size-limit
#    size: 256mb
#  strategy:
#    type: numeric
#    max: 10
#  layout:
#    type: json

# Logs queries sent to Elasticsearch.
#logging.loggers:
#  - name: elasticsearch.query
#    level: debug

# Logs http responses.
#logging.loggers:
#  - name: http.server.response
#    level: debug

# Logs system usage information.
#logging.loggers:
#  - name: metrics.ops
#    level: debug

# Enables debug logging on the browser (dev console)
#logging.browser.root:
#  level: debug

# =================== System: Other ===================
# The path where Kibana stores persistent data not saved in Elasticsearch. Defaults to data
#path.data: data

# Specifies the path where Kibana creates the process ID file.
#pid.file: /run/kibana/kibana.pid

# Set the interval in milliseconds to sample system and process performance
# metrics. Minimum is 100ms. Defaults to 5000ms.
#ops.interval: 5000

# Specifies locale to be used for all localizable strings, dates and number formats.
# Supported languages are the following: English (default) "en", Chinese "zh-CN", Japanese "ja-JP", French "fr-FR".
#i18n.locale: "en"

# =================== Frequently used (Optional)===================

# =================== Saved Objects: Migrations ===================
# Saved object migrations run at startup. If you run into migration-related issues, you might need to adjust these settings.

# The number of documents migrated at a time.
# If Kibana can't start up or upgrade due to an Elasticsearch `circuit_breaking_exception`,
# use a smaller batchSize value to reduce the memory pressure. Defaults to 1000 objects per batch.
#migrations.batchSize: 1000

# The maximum payload size for indexing batches of upgraded saved objects.
# To avoid migrations failing due to a 413 Request Entity Too Large response from Elasticsearch.
# This value should be lower than or equal to your Elasticsearch cluster’s `http.max_content_length`
# configuration option. Default: 100mb
#migrations.maxBatchSizeBytes: 100mb

# The number of times to retry temporary migration failures. Increase the setting
# if migrations fail frequently with a message such as `Unable to complete the [...] step after
# 15 attempts, terminating`. Defaults to 15
#migrations.retryAttempts: 15

# =================== Search Autocomplete ===================
# Time in milliseconds to wait for autocomplete suggestions from Elasticsearch.
# This value must be a whole number greater than zero. Defaults to 1000ms
#unifiedSearch.autocomplete.valueSuggestions.timeout: 1000

# Maximum number of documents loaded by each shard to generate autocomplete suggestions.
# This value must be a whole number greater than zero. Defaults to 100_000
#unifiedSearch.autocomplete.valueSuggestions.terminateAfter: 100000

```

**Step 7: Playbook Execution**

* To set up Kibanas on your target servers, you will execute the Ansible playbook using the following command:

```bash
ansible-playbook -i aws_ec2.yml kibana_playbook.yml
```

> Additional Options
> 
> --limit: You can use this option to specify a subset of hosts from the inventory on which the playbook should be executed.
> 
> -e or --extra-vars: You can pass extra variables to the playbook using this option.


***
## Output
**Host-level output**: Output for each host would indicate whether the playbook execution was successful or not.

![Screenshot 2024-04-07 155342](https://github.com/CodeOps-Hub/Documentation/assets/156056344/7d76ab9e-91d6-4af8-bd79-135ed5fdf850)


***

## Post-Installation Setup
* Access Kibana on `http://URL:5601`
  
![Screenshot 2024-04-07 155322](https://github.com/CodeOps-Hub/Documentation/assets/156056344/12509da0-2431-4499-8275-bac5a5f4f929)


***
## Conclusion 

* This guide illustrates the process of deploying Kibana through Ansible. By adhering to these instructions, you can effectively provision and set up redis within your AWS infrastructure.

***
## Contact Information

|Aakash Tripathi                 | aakash.tripathi.snaatak@mygurukulam.co                                                                                      
|---------------------------------|------------------------------------------------------------|

***
## References

| Title                                      | URL                                           |
|--------------------------------------------|-----------------------------------------------|
| Ansible documentation           | https://docs.ansible.com/ansible/latest/index.html    |
| Kibana Installation          | https://www.elastic.co/guide/en/kibana/current/deb.html |
| Kibana Default Configurations | https://github.com/elastic/kibana/blob/main/config/kibana.yml |
| Dyanmic Inventory               | https://www.youtube.com/watch?v=junPdh2yvbU&t=454s | 









