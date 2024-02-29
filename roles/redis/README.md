# Ansible Role to setup Redis
![download (1)](https://github.com/CodeOps-Hub/redis-ansiblerole/assets/156056344/88c00e2a-36c4-4087-95e0-c83833747845)



|   Authors        |  Created on   |  Version   | Last updated by | Last edited on |
| -----------------| --------------| -----------|---------------- | -------------- |
| Aakash Tripathi      |  19 Feb 2024   |     v1     | Aakash Tripathi     | 19 Feb 2024    |

***
## Table of Contents
+ [Introduction](#Introduction)
+ [Flow Diagram](#flow-diagram)
+ [Pre-requisites](#pre-requisites)
+ [Setup Ansible Role](#steps)
+ [Output Verification](#output)
+ [Connect Redis](#post-installation-steps)
+ [Conclusion](#conclusion)
+ [Contact Information](#contact-information)
+ [References](#references)

***
## Introduction
This role is designed to automate the installation and configuration of Redis on target ubuntu servers. This role aims to simplify the process of setting up standalone redis servers.

***
## Flow Diagram

* This diagram should help you visualize the sequence of tasks in the Ansible role for setting up Redis.
![Screenshot 2024-02-19 221431](https://github.com/CodeOps-Hub/redis-ansiblerole/assets/156056344/7545e86d-43be-4571-b9b4-ca2e02bf2b00)


***
## Pre-requisites

Before using this Ansible role to set up Redis, ensure that the following prerequisites are met:

1. **Ansible:**
   - Ansible must be installed on the control machine from which you plan to run the playbook. If Ansible is not installed, you can install it using this [link](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) . Version used for POC : ansible 2.10.8


### Ansible 
![ansible_logo_icon_169596](https://github.com/avengers-p7/Documentation/assets/156056344/21281851-6cfa-4b18-aee4-8812e193dc62)

Ansible is an open-source automation tool that simplifies and accelerates IT infrastructure management, application deployment, and task automation. Employing a declarative language, Ansible enables users to define desired states for systems and applications, automating complex workflows efficiently. With agentless architecture, it connects to remote systems over SSH or other protocols, making it versatile and easy to implement. 


2. **SSH Access to Target Servers:**
   - Ensure that you have SSH access to the target servers where Redis will be installed.

***

## Redis  
Redis is an open-source, high-performance, in-memory data store known for its speed and versatility. It functions as a key-value store and supports various data structures, including strings, hashes, lists, sets, and sorted sets. Commonly used for caching due to its in-memory storage, Redis also serves as a message broker with Pub/Sub messaging capabilities. Its atomic operations, support for persistence, scripting with Lua, and clustering features make Redis a popular choice for applications requiring fast data access, real-time processing, and horizontal scalability.

Please refer [*Redis Document*](https://github.com/avengers-p7/Documentation/blob/main/OT%20Micro%20Services/Software/Redis/README.md) for better understanding of Redis

# Steps 
* Before going further check  [*Ansible Role For Redis Installation*](https://github.com/CodeOps-Hub/redis-ansiblerole/tree/main/roles/redis)
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
private_key_file = /path/to/private_key

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
  - your_aws_region

groups: 
  redis: "'redis' in tags.Type"
```

1. `plugin: aws_ec2`: Specifies the use of the aws_ec2 plugin as the dynamic inventory source. This plugin is designed to fetch information about EC2 instances in AWS.
2. `regions: - us-east-1`: Indicates the AWS region(s) from which the dynamic inventory should fetch information.
3. `redis: "'redis' in tags.Type"`: Creates an Ansible group named redis. This group includes EC2 instances where the tag named Type has a value of 'redis'. You can tag all your redis instances accordingly.

**Step 3: Create Ansible Role**
* Create a new Ansible role which should follow this directory structure:

![Screenshot 2024-02-19 213938](https://github.com/CodeOps-Hub/redis-ansiblerole/assets/156056344/5ddada64-d834-4f36-8bc4-9638ff6e0c0e)



**Step 4: playbook.yml**
* This file is defining a set of tasks to be executed on hosts belonging to the ubuntu group.

```yaml
---
- hosts: redis
  become: yes
  gather_facts: yes 
  roles:
    - redis
```
**Step 5: Tasks**
1. `main.yml`: This main.yml file is acting as an orchestrator, importing tasks from other task files. This separation of tasks into different files is a good practice for better organization, especially when dealing with complex configurations or roles.

```yaml
---
---
# tasks file for redis

- name: Ensuring prerequisites are met for redis installation
  include_tasks: prerequisites.yml

- name: Ensuring redis server is installed on the system
  include_tasks: install.yml

- name: Ensuring redis configuration is up to date
  include_tasks: configure.yml
```

2. `Default` variables: This role comes with default values for several variables that have been used in the role. You can find these defaults in the `defaults/main.yml` file within the role directory.

```yaml
---
# defaults file for redis

upgrade_redis: false
redis_version: 7.0.6
redis_port: 6379
redis_conf_file_location: /etc/redis
redis_data_dir: /opt/redis/data
redis_log_dir: /var/log/redis
redis_logfile: "/var/log/redis/redis.log"

redis_supervised: "no"
redis_always_show_logo: "yes"
redis_daemonize: "no"
redis_pidfile: "/var/run/redis-{{- redis_port -}}.pid"
redis_databases: 16
redis_loglevel: notice
redis_save:
  - 900 1
  - 300 10
  - 60 10000
redis_stop_writes_on_bgsave_error: "yes"
redis_rdbcompression: "yes"
redis_rdbchecksum: "yes"
redis_db_filename: "dump.rdb"
redis_slowlog_log_slower_than: 10000
redis_slowlog_max_len: 128
redis_appendonly: "no"
redis_appendfilename: "appendonly.aof"
redis_appendfsync: "everysec"
redis_no_appendfsync_on_rewrite: "no"
redis_auto_aof_rewrite_percentage: "100"
redis_auto_aof_rewrite_min_size: "64mb"
redis_maxclients: 10000
redis_maxmemory: false
redis_maxmemory_policy: noeviction
redis_bind: "{{ ansible_ssh_host }}"
redis_tcp_backlog: 511
redis_tcp_keepalive: 0
redis_protected_mode: "no"
redis_timeout: 0
redis_socket_path: false
redis_socket_perm: 755
redis_conf_file_location: /etc/redis
redis_runtime_directory: redis
disable_commands:
  - FLUSHDB
  - FLUSHALL
```
### Important Role Variables 
| **Variable** | **Description** |
| ------------ | --------------- |
| redis_version |  Version of redis to be installed |
| redis_port | Default redis port |
| redis_conf_file_location | Redis configuration file directory path |
| redis_data_dir | Path to redis data storage directory |
| redis_log_dir | Redis logs storage directory |
| redis_logfile | Redis logfile |
| redis_password | Redis Password |

> [!NOTE]
> To customize the Redis installation based on your specific requirements, you can override these default values in main.yaml file in the vars directory of the role. 


3. `prerequisites.yml`: This file is included in the redis/tasks/main.yml file to make sure prerequisites are met before redis installation

```yaml
---
- name: Ensure build tools are installed on the system (Debian)
  apt:
    name: "build-essential"
    state: present
    update_cache: yes
  become: yes
  when: ansible_os_family == "Debian"

- name: Ensure build tools are installed on the system (RedHat)
  yum:
    name: "make"
    state: present
  become: yes
  when: ansible_os_family == "RedHat"

- name: Ensuring redis performance parameters are defined
  sysctl:
    name: vm.overcommit_memory
    value: "1"
    state: present
  become: yes

- name: Ensuring the maximum number of connection count is set
  sysctl:
    name: net.core.somaxconn
    value: "65365"
    state: present
  become: yes

- name: Ensure that transparent huge page is disabled in Kernel
  shell: "echo never > /sys/kernel/mm/transparent_hugepage/enabled"
  become: yes
  become_user: root
  changed_when: False
```

4. `install.yml`: This file is included in the redis/tasks/main.yml file to perform redis installation
 ```yaml
 ---
- name: Ensure if redis is previously installed or not
  stat:
    path: "/usr/local/bin/redis-server"
  register: redis_status

- name: Ensure that the defined redis version is downloaded {{ redis_version }}
  unarchive:
    src: "http://download.redis.io/releases/redis-{{ redis_version }}.tar.gz"
    dest: /tmp/
    remote_src: yes
  when: >
    not redis_status.stat.exists or
    upgrade_redis

- name: Ensure the build time dependencies are met
  make:
    chdir: "/tmp/redis-{{ redis_version }}"
  when: >
    not redis_status.stat.exists or
    upgrade_redis
  become: yes

- name: Ensuring redis server is installed on the system
  make:
    chdir: "/tmp/redis-{{ redis_version }}"
    target: install
  when: >
    not redis_status.stat.exists or
    upgrade_redis
  become: yes

- name: Ensure the tarballs and build directory is cleaned up
  file:
    path: "{{ item }}"
    state: absent
  with_items:
    - "/tmp/redis-{{ redis_version }}.tar.gz"
    - "/tmp/redis-{{ redis_version }}"
  when: >
    not redis_status.stat.exists or
    upgrade_redis
  become: yes

- name: Ensuring that redis group exists in system
  group:
    name: "redis"
    state: present
  become: yes

- name: Ensuring that redis user exists in system
  user:
    name: "redis"
    group: "redis"
    shell: /sbin/nologin
    password: "!!"
    createhome: no
    system: yes
    state: present
  become: yes
 ```
 5.  `configure.yml` : This file is included in the redis/tasks/main.yml file to configure redis, redis service and start redis service
   ```yaml
      ---
    - name: Ensure the configuration directories are present
    file:
      path: "{{ item }}"
      state: directory
      recurse: yes
      owner: redis
      group: redis
    with_items:
      - "{{ redis_log_dir }}"
      - "{{ redis_data_dir }}"
      - "{{ redis_conf_file_location }}"
    become: yes
  
    - name: Ensure that redis service exists on the system
    template:
      src: redis.service.j2
      dest: "/etc/systemd/system/redis.service"
      owner: root
      group: root
    become: yes
    notify:
      - Ensure system daemon reloaded
  
    - name: Ensure the redis configuration is updated
    template:
      src: redis.conf.j2
      dest: "{{ redis_conf_file_location }}/redis.conf"
      owner: redis
      group: redis
    become: yes
    notify:
      - Ensure redis service is running

   ```


**Step 6: Templates for Configuration**
We need to create two jinja2 templates :
* To configure Redis
* To set up redis Service

1. `redis.conf.j2` template includes parameteters to configure Redis

```yaml
{{ ansible_managed | comment }}
#network
bind 0.0.0.0
protected-mode {{ redis_protected_mode }}
port {{ redis_port }}
tcp-keepalive {{ redis_tcp_keepalive }}
tcp-backlog {{ redis_tcp_backlog }}
timeout {{ redis_timeout }}
{% if redis_socket_path -%}
unixsocket {{ redis_socket_path }}
unixsocketperm {{ redis_socket_perm }}
{% endif -%}

# General
supervised {{redis_supervised}}
always-show-logo {{redis_always_show_logo}}
daemonize {{ redis_daemonize }}
pidfile {{ redis_pidfile }}
loglevel {{ redis_loglevel }}
databases {{ redis_databases }}
logfile {{ redis_logfile }}

# Snapshotting
{% for save in redis_save %}
save {{ save }}
{% endfor %}
stop-writes-on-bgsave-error {{ redis_stop_writes_on_bgsave_error|string }}
rdbcompression {{ redis_rdbcompression|string }}
rdbchecksum {{ redis_rdbchecksum|string }}
dbfilename {{ redis_db_filename|string }}
dir {{ redis_data_dir }}

# Disable Redis commands
{% for disable_command in disable_commands %}
rename-command {{ disable_command }} ""
{% endfor %}

# Clients -- not in default redis.conf
maxclients {{ redis_maxclients }}

#Memory Management
maxmemory-policy {{ redis_maxmemory_policy }}
{% if redis_maxmemory -%}
maxmemory {{ redis_maxmemory }}
{%- endif %}
# Append Only Mode
appendonly {{ redis_appendonly }}
appendfilename "{{ redis_appendfilename }}"
appendfsync {{ redis_appendfsync|string }}
no-appendfsync-on-rewrite {{ redis_no_appendfsync_on_rewrite }}
auto-aof-rewrite-percentage {{ redis_auto_aof_rewrite_percentage }}
auto-aof-rewrite-min-size {{ redis_auto_aof_rewrite_min_size }}


# Slow Log
slowlog-log-slower-than {{ redis_slowlog_log_slower_than }}
slowlog-max-len {{ redis_slowlog_max_len }}

# latency monitor
latency-monitor-threshold 0

{% if redis_password -%}
requirepass {{ redis_password }}
{%- endif %}
```

2. `redis.service.j2` template creates a service file for setting up `redis.service`
```ini
[Unit]
Description=Redis persistent key-value database
After=network.target

[Service]
ExecStart=/usr/local/bin/redis-server {{ redis_conf_file_location }}/redis.conf --supervised systemd
ExecStop=/bin/redis-cli shutdown
Type=simple
User=redis
Group=redis
RuntimeDirectory={{ redis_runtime_directory }}
RuntimeDirectoryMode=0755
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
```

**Step 7: Playbook Execution**

* To set up Redis on your target servers, you will execute the Ansible playbook using the following command:

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

![Screenshot 2024-02-19 202326](https://github.com/CodeOps-Hub/redis-ansiblerole/assets/156056344/e4b974a2-66f5-4e86-896f-1ef36e172cf1)

***

## Post-Installation Setup
* Connect to redis using `redis-cli`
  
   ![Screenshot 2024-02-19 220233](https://github.com/CodeOps-Hub/redis-ansiblerole/assets/156056344/1d42ae6e-b031-4fb7-b156-6999ce9da7d0)

***
## Conclusion 

* This guide illustrates the process of deploying redis through Ansible. By adhering to these instructions, you can effectively provision and set up redis within your AWS infrastructure.

***
## Contact Information

|Aakash Tripathi                 | aakash.tripathi.snaatak@mygurukulam.co                                                                                      
|---------------------------------|------------------------------------------------------------|

***
## References

| Title                                      | URL                                           |
|--------------------------------------------|-----------------------------------------------|
| Ansible documentation           | https://docs.ansible.com/ansible/latest/index.html    |
| Redis Installation          | https://redis.io/docs/install/install-redis/install-redis-from-source/ |
| Dyanmic Inventory               | https://www.youtube.com/watch?v=junPdh2yvbU&t=454s | 








