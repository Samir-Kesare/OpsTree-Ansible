# Ansible Role for Attendance-Grafana-Dashboard
![image](https://github.com/CodeOps-Hub/Ansible/assets/156056444/499089ea-786d-4975-b37a-7b851fb16941)

***
|   Author        |  Created on   |  Version   | Last updated by  | Last edited on |
| --------------- | --------------| -----------|----------------- | -------------- |
| **[Harshit Singh](https://github.com/Panu-S-Harshit-Ninja-07)**    | 10 April 2024 |  Version 1 | Harshit Singh     | 10 April 2024  |
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

This role is designed to automate the procees of adding Attendance Grafana Dashboard on target PGA server. This document aims to simplify the process and explain the role created and used for setting up of Attendance Grafana Dashboard.

***

# Flow Diagram

![image](https://github.com/CodeOps-Hub/Ansible/assets/156056444/1286ae5d-6020-4688-a48c-aa271a8e9d2a)

***

# Pre-requisites

| **Pre-requisite** | **Description** |
| ----------------- | --------------- |
| **Ansible**       | Ansible must be installed on the control machine from which you plan to run the playbook. |
| **AWS CLI**       | AWS CLI for providing aws credentials to fetch resources. |
| **Python**        | Ensure that Python is installed on your system. Ansible relies on Python for its execution, and dynamic inventory scripts are typically written in Python. |
| **PIP (Python Package Installer)** | Install pip if it's not already installed. pip is a package manager for Python that allows you to install and manage Python packages. |
| **Boto3**   |  If your dynamic inventory script relies on AWS APIs to fetch inventory data, you'll need to install `boto3` using `pip`. |
| **PGA Server** | Must have installed `prometheus, grafana and alert manager(optional)` with `security group` having necessary ports allowed on it. |

***

# Directory Structure
![image](https://github.com/CodeOps-Hub/Ansible/assets/156056444/c46be773-3869-4ad5-9cfa-b4d840b58bb3)

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

# some basic default values...


# Use AWS EC2 dynamic inventory for managing hosts
inventory      = aws_ec2.yaml

# Disable SSH host key checking for convenience.
host_key_checking = False

# Specify the path to the private key file for SSH connections.
private_key_file = /home/harshit/Downloads/snaatak.pem 

#Sets the remote user for SSH connections to 'ubuntu'
remote_user = ubuntu

[inventory]
# enable inventory plugins, default: 'host_list', 'script', 'auto', 'yaml', 'ini', 'toml'
enable_plugins = aws_ec2, host_list, virtualbox, yaml, constructed, script, auto, ini, toml

```
</details>

***

### aws_ec2.yaml inventory file

<details>
<summary> Click here to see aws_ec2.yml file</summary>
<br>
  
```shell
---
plugin: aws_ec2
regions:
  - us-east-1

groups: 
  redis: "'redis' in tags.Type"
  postgres: "'postgres' in tags.Type"
  scylla: "'scylla' in tags.Type"
  efk: "'efk' in tags.Type"
  prometheus: "'prometheus' in tags.Type"
  frontend: "'frontend-node-exporter' in tags.Type"
  employee: "'employee' in tags.Type"
  pga: "'pga' in tags.Type"

filters:
    instance-state-code: 16

```
</details>

***

### playbook.yml file

<details>
<summary> Click here to see playbook.yml file</summary>
<br>
  
```shell
---
- hosts: pga
  become: yes
  gather_facts: yes
  roles:
    - attendanceGfDashboard

```
</details>

***

## Salary-Node-Exporter Role

### addDashboard file (addDashboard.yml)
This Ansible playbook consists of tasks aimed at copying json file , adding dashboard in Grafana then deleting that json file.
<details>
<summary> Click here to see ubuntu.yml file</summary>
<br>
  
```shell
---
# tasks file for attendanceGfDashboard

- name: Import json file template
  ansible.builtin.template:
    src: "{{ json_file_template_src }}"
    dest: "{{ json_file_dest }}"

- name: Import Attendance Grafana dashboard 
  community.grafana.grafana_dashboard:
    grafana_url: "{{ grafana_url }}"
    grafana_api_key: "{{ grafana_api_key }}"
    state: present
    commit_message: "Updated by {{ editor_name }}"
    overwrite: true
    path: "{{ json_file_dest }}"

- name: Remove json file (delete file)
  ansible.builtin.file:
    path: "{{ json_file_dest }}"
    state: absent
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
# tasks file for attendanceGfDashboard
- name: Include tasks for Attendance 
  ansible.builtin.include_tasks: addDashboard.yml

```
</details>

***

### templates file (node_exporter_service.j2)

This configuration sets up a systemd service for Prometheus Node Exporter tailored for Salary App monitoring. It ensures that the service starts after the network is available. The service runs as the user and group "node_exporter" and launches Node Exporter with Salary-specific collectors enabled, listening on the specified address and port. Finally, it specifies that the service should be enabled and started during the multi-user boot sequence.

<details>
<summary> Click here to see node_exporter_service.j2 file</summary>
<br>
  
```shell
{
  "__inputs": [
    {
      "name": "P7-PROMETHEUS",
      "label": "p7-prometheus",
      "description": "",
      "type": "datasource",
      "pluginId": "prometheus",
      "pluginName": "Prometheus"
    }
  ],
  "__elements": {},
  "__requires": [
    {
      "type": "panel",
      "id": "bargauge",
      "name": "Bar gauge",
      "version": ""
    },
    {
      "type": "panel",
      "id": "gauge",
      "name": "Gauge",
      "version": ""
    },
    {
      "type": "grafana",
      "id": "grafana",
      "name": "Grafana",
      "version": "10.4.1"
    },
    {
      "type": "datasource",
      "id": "prometheus",
      "name": "Prometheus",
      "version": "1.0.0"
    },
    {
      "type": "panel",
      "id": "stat",
      "name": "Stat",
      "version": ""
    },
    {
      "type": "panel",
      "id": "timeseries",
      "name": "Time series",
      "version": ""
    }
  ],
  "annotations": {
    "list": [
      {
        "$$hashKey": "object:1058",
        "builtIn": 1,
        "datasource": {
          "type": "datasource",
          "uid": "grafana"
        },
        "enable": true,
        "hide": true,
        "iconColor": "rgba(0, 211, 255, 1)",
        "name": "Annotations & Alerts",
        "target": {
          "limit": 100,
          "matchAny": false,
          "tags": [],
          "type": "dashboard"
        },
        "type": "dashboard"
      }
    ]
  },
  "editable": true,
  "fiscalYearStartMonth": 0,
  "gnetId": 1860,
  "graphTooltip": 1,
  "id": null,
  "links": [
    {
      "icon": "external link",
      "tags": [],
      "targetBlank": true,
      "title": "GitHub",
      "type": "link",
      "url": "https://github.com/rfmoz/grafana-dashboards"
    },
    {
      "icon": "external link",
      "tags": [],
      "targetBlank": true,
      "title": "Grafana",
      "type": "link",
      "url": "https://grafana.com/grafana/dashboards/1860"
    }
  ],
  "liveNow": false,
  "panels": [
    {
      "collapsed": false,
      "gridPos": {
        "h": 1,
        "w": 24,
        "x": 0,
        "y": 0
      },
      "id": 330,
      "panels": [],
      "title": "Attendance API",
      "type": "row"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "{{ prometheus_DS_uid }}"
      },
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "thresholds"
          },
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "#bb77dcfa",
                "value": null
              }
            ]
          }
        },
        "overrides": []
      },
      "gridPos": {
        "h": 5,
        "w": 4,
        "x": 0,
        "y": 1
      },
      "id": 325,
      "options": {
        "displayMode": "gradient",
        "maxVizHeight": 300,
        "minVizHeight": 16,
        "minVizWidth": 8,
        "namePlacement": "auto",
        "orientation": "auto",
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "showUnfilled": true,
        "sizing": "auto",
        "valueMode": "color"
      },
      "pluginVersion": "10.4.1",
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "{{ prometheus_DS_uid }}"
          },
          "editorMode": "code",
          "expr": "sum by (method, path) (flask_http_request_total)\n",
          "instant": false,
          "legendFormat": "__auto",
          "range": true,
          "refId": "A"
        }
      ],
      "title": "TOTAL REQUEST",
      "type": "bargauge"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "{{ prometheus_DS_uid }}"
      },
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "palette-classic"
          },
          "custom": {
            "axisBorderShow": false,
            "axisCenteredZero": false,
            "axisColorMode": "text",
            "axisLabel": "",
            "axisPlacement": "auto",
            "barAlignment": 0,
            "drawStyle": "line",
            "fillOpacity": 10,
            "gradientMode": "hue",
            "hideFrom": {
              "legend": false,
              "tooltip": false,
              "viz": false
            },
            "insertNulls": false,
            "lineInterpolation": "linear",
            "lineWidth": 1,
            "pointSize": 5,
            "scaleDistribution": {
              "type": "linear"
            },
            "showPoints": "auto",
            "spanNulls": false,
            "stacking": {
              "group": "A",
              "mode": "none"
            },
            "thresholdsStyle": {
              "mode": "off"
            }
          },
          "mappings": [],
          "thresholds": {
            "mode": "percentage",
            "steps": [
              {
                "color": "green",
                "value": null
              },
              {
                "color": "red",
                "value": 80
              }
            ]
          }
        },
        "overrides": []
      },
      "gridPos": {
        "h": 6,
        "w": 5,
        "x": 4,
        "y": 1
      },
      "id": 329,
      "options": {
        "legend": {
          "calcs": [],
          "displayMode": "list",
          "placement": "bottom",
          "showLegend": true
        },
        "tooltip": {
          "mode": "single",
          "sort": "none"
        }
      },
      "pluginVersion": "10.4.1",
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "{{ prometheus_DS_uid }}"
          },
          "editorMode": "code",
          "expr": "rate(flask_http_request_duration_seconds_sum{method=\"GET\", path=\"/api/v1/attendance/health\", status=\"200\"}[5m]) / rate(flask_http_request_duration_seconds_count{method=\"GET\", path=\"/api/v1/attendance/health\", status=\"200\"}[5m]) \n",
          "instant": false,
          "legendFormat": "__auto",
          "range": true,
          "refId": "A"
        }
      ],
      "title": "API Health",
      "type": "timeseries"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "{{ prometheus_DS_uid }}"
      },
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "palette-classic"
          },
          "custom": {
            "axisBorderShow": false,
            "axisCenteredZero": false,
            "axisColorMode": "text",
            "axisLabel": "",
            "axisPlacement": "auto",
            "barAlignment": 0,
            "drawStyle": "line",
            "fillOpacity": 15,
            "gradientMode": "opacity",
            "hideFrom": {
              "legend": false,
              "tooltip": false,
              "viz": false
            },
            "insertNulls": false,
            "lineInterpolation": "linear",
            "lineStyle": {
              "fill": "solid"
            },
            "lineWidth": 1,
            "pointSize": 5,
            "scaleDistribution": {
              "type": "linear"
            },
            "showPoints": "auto",
            "spanNulls": false,
            "stacking": {
              "group": "A",
              "mode": "none"
            },
            "thresholdsStyle": {
              "mode": "off"
            }
          },
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "blue",
                "value": null
              }
            ]
          }
        },
        "overrides": [
          {
            "__systemRef": "hideSeriesFrom",
            "matcher": {
              "id": "byNames",
              "options": {
                "mode": "exclude",
                "names": [
                  "sum(rate(flask_http_request_duration_seconds_sum{method=\"GET\", status=\"200\", instance=\"dev-alb-590116793.us-east-1.elb.amazonaws.com:80\"}[5m])) / sum(rate(flask_http_request_duration_seconds_count{method=\"GET\", status=\"200\", instance=\"dev-alb-590116793.us-east-1.elb.amazonaws.com:80\"}[5m])) \n"
                ],
                "prefix": "All except:",
                "readOnly": true
              }
            },
            "properties": [
              {
                "id": "custom.hideFrom",
                "value": {
                  "legend": false,
                  "tooltip": false,
                  "viz": true
                }
              }
            ]
          }
        ]
      },
      "gridPos": {
        "h": 7,
        "w": 6,
        "x": 9,
        "y": 1
      },
      "id": 324,
      "options": {
        "legend": {
          "calcs": [],
          "displayMode": "list",
          "placement": "bottom",
          "showLegend": true
        },
        "tooltip": {
          "mode": "single",
          "sort": "none"
        }
      },
      "pluginVersion": "10.4.1",
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "{{ prometheus_DS_uid }}"
          },
          "editorMode": "code",
          "expr": "sum(rate(flask_http_request_duration_seconds_sum{method=\"GET\", status=\"200\", instance=\"dev-alb-590116793.us-east-1.elb.amazonaws.com:80\"}[5m])) / sum(rate(flask_http_request_duration_seconds_count{method=\"GET\", status=\"200\", instance=\"dev-alb-590116793.us-east-1.elb.amazonaws.com:80\"}[5m])) \n",
          "instant": false,
          "legendFormat": "__auto",
          "range": true,
          "refId": "A"
        }
      ],
      "title": "SUCCESS RATE",
      "type": "timeseries"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "{{ prometheus_DS_uid }}"
      },
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "palette-classic"
          },
          "custom": {
            "axisBorderShow": false,
            "axisCenteredZero": false,
            "axisColorMode": "text",
            "axisLabel": "",
            "axisPlacement": "auto",
            "barAlignment": 0,
            "drawStyle": "line",
            "fillOpacity": 0,
            "gradientMode": "none",
            "hideFrom": {
              "legend": false,
              "tooltip": false,
              "viz": false
            },
            "insertNulls": false,
            "lineInterpolation": "linear",
            "lineWidth": 1,
            "pointSize": 5,
            "scaleDistribution": {
              "type": "linear"
            },
            "showPoints": "auto",
            "spanNulls": false,
            "stacking": {
              "group": "A",
              "mode": "none"
            },
            "thresholdsStyle": {
              "mode": "off"
            }
          },
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              },
              {
                "color": "red",
                "value": 80
              }
            ]
          }
        },
        "overrides": [
          {
            "matcher": {
              "id": "byName",
              "options": "sum(rate(flask_http_request_duration_seconds_sum{method=\"GET\", status=\"404\", instance=\"dev-alb-590116793.us-east-1.elb.amazonaws.com:80\"}[5m])) / sum(rate(flask_http_request_duration_seconds_count{method=\"GET\", status=\"404\", instance=\"dev-alb-590116793.us-east-1.elb.amazonaws.com:80\"}[5m])) \n"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "dark-red",
                  "mode": "fixed"
                }
              }
            ]
          },
          {
            "__systemRef": "hideSeriesFrom",
            "matcher": {
              "id": "byNames",
              "options": {
                "mode": "exclude",
                "names": [
                  "sum(rate(flask_http_request_duration_seconds_sum{method=\"GET\", status=\"404\", instance=\"dev-alb-590116793.us-east-1.elb.amazonaws.com:80\"}[5m])) / sum(rate(flask_http_request_duration_seconds_count{method=\"GET\", status=\"404\", instance=\"dev-alb-590116793.us-east-1.elb.amazonaws.com:80\"}[5m])) \n"
                ],
                "prefix": "All except:",
                "readOnly": true
              }
            },
            "properties": [
              {
                "id": "custom.hideFrom",
                "value": {
                  "legend": false,
                  "tooltip": false,
                  "viz": true
                }
              }
            ]
          }
        ]
      },
      "gridPos": {
        "h": 7,
        "w": 6,
        "x": 15,
        "y": 1
      },
      "id": 328,
      "options": {
        "legend": {
          "calcs": [],
          "displayMode": "list",
          "placement": "bottom",
          "showLegend": true
        },
        "tooltip": {
          "mode": "single",
          "sort": "none"
        }
      },
      "pluginVersion": "10.4.1",
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "{{ prometheus_DS_uid }}"
          },
          "editorMode": "code",
          "expr": "sum(rate(flask_http_request_duration_seconds_sum{method=\"GET\", status=\"404\", instance=\"dev-alb-590116793.us-east-1.elb.amazonaws.com:80\"}[5m])) / sum(rate(flask_http_request_duration_seconds_count{method=\"GET\", status=\"404\", instance=\"dev-alb-590116793.us-east-1.elb.amazonaws.com:80\"}[5m])) \n",
          "instant": false,
          "legendFormat": "__auto",
          "range": true,
          "refId": "A"
        }
      ],
      "title": "ERROR RATE",
      "type": "timeseries"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "{{ prometheus_DS_uid }}"
      },
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "thresholds"
          },
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "yellow",
                "value": null
              },
              {
                "color": "red",
                "value": 80
              }
            ]
          }
        },
        "overrides": []
      },
      "gridPos": {
        "h": 5,
        "w": 3,
        "x": 21,
        "y": 1
      },
      "id": 326,
      "options": {
        "colorMode": "value",
        "graphMode": "area",
        "justifyMode": "auto",
        "orientation": "auto",
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "showPercentChange": false,
        "textMode": "auto",
        "wideLayout": true
      },
      "pluginVersion": "10.4.1",
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "{{ prometheus_DS_uid }}"
          },
          "editorMode": "code",
          "expr": "sum(flask_http_request_exceptions_total)",
          "instant": false,
          "legendFormat": "__auto",
          "range": true,
          "refId": "A"
        }
      ],
      "title": "Number of Exceptions",
      "type": "stat"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "{{ prometheus_DS_uid }}"
      },
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "thresholds"
          },
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "dark-orange",
                "value": null
              },
              {
                "color": "dark-red",
                "value": 80
              }
            ]
          }
        },
        "overrides": [
          {
            "matcher": {
              "id": "byName",
              "options": "sum(rate(flask_http_request_duration_seconds_sum{method=\"GET\", status!=\"200\", instance=\"dev-alb-590116793.us-east-1.elb.amazonaws.com:80\"}[5m])) / sum(rate(flask_http_request_duration_seconds_count{method=\"GET\", status!=\"200\", instance=\"dev-alb-590116793.us-east-1.elb.amazonaws.com:80\"}[5m])) \n"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "red",
                  "mode": "fixed"
                }
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "404"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "red",
                  "mode": "fixed"
                }
              }
            ]
          }
        ]
      },
      "gridPos": {
        "h": 5,
        "w": 4,
        "x": 0,
        "y": 6
      },
      "id": 327,
      "options": {
        "colorMode": "value",
        "graphMode": "area",
        "justifyMode": "auto",
        "orientation": "auto",
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "showPercentChange": false,
        "textMode": "auto",
        "wideLayout": true
      },
      "pluginVersion": "10.4.1",
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "{{ prometheus_DS_uid }}"
          },
          "editorMode": "code",
          "expr": "sum by (status) (flask_http_request_total{status=\"404\"})\n",
          "instant": false,
          "legendFormat": "__auto",
          "range": true,
          "refId": "A"
        }
      ],
      "title": "404 ERRORS",
      "type": "stat"
    },
    {
      "collapsed": false,
      "datasource": {
        "type": "prometheus",
        "uid": "000000001"
      },
      "gridPos": {
        "h": 1,
        "w": 24,
        "x": 0,
        "y": 11
      },
      "id": 261,
      "panels": [],
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "000000001"
          },
          "refId": "A"
        }
      ],
      "title": "Quick CPU / Mem / Disk",
      "type": "row"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "{{ prometheus_DS_uid }}"
      },
      "description": "Resource pressure via PSI",
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "thresholds"
          },
          "decimals": 1,
          "links": [],
          "mappings": [],
          "max": 1,
          "min": 0,
          "thresholds": {
            "mode": "percentage",
            "steps": [
              {
                "color": "green",
                "value": null
              },
              {
                "color": "dark-yellow",
                "value": 70
              },
              {
                "color": "dark-red",
                "value": 90
              }
            ]
          },
          "unit": "percentunit"
        },
        "overrides": []
      },
      "gridPos": {
        "h": 4,
        "w": 3,
        "x": 0,
        "y": 12
      },
      "id": 323,
      "options": {
        "displayMode": "basic",
        "maxVizHeight": 300,
        "minVizHeight": 10,
        "minVizWidth": 0,
        "namePlacement": "auto",
        "orientation": "horizontal",
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "showUnfilled": true,
        "sizing": "auto",
        "text": {},
        "valueMode": "color"
      },
      "pluginVersion": "10.4.1",
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "{{ prometheus_DS_uid }}"
          },
          "editorMode": "code",
          "exemplar": false,
          "expr": "irate(node_pressure_cpu_waiting_seconds_total{instance=\"$node\",job=\"$job\"}[$__rate_interval])",
          "format": "time_series",
          "instant": true,
          "intervalFactor": 1,
          "legendFormat": "CPU",
          "range": false,
          "refId": "CPU some",
          "step": 240
        },
        {
          "datasource": {
            "type": "prometheus",
            "uid": "{{ prometheus_DS_uid }}"
          },
          "editorMode": "code",
          "exemplar": false,
          "expr": "irate(node_pressure_memory_waiting_seconds_total{instance=\"$node\",job=\"$job\"}[$__rate_interval])",
          "format": "time_series",
          "hide": false,
          "instant": true,
          "intervalFactor": 1,
          "legendFormat": "Mem",
          "range": false,
          "refId": "Memory some",
          "step": 240
        },
        {
          "datasource": {
            "type": "prometheus",
            "uid": "{{ prometheus_DS_uid }}"
          },
          "editorMode": "code",
          "exemplar": false,
          "expr": "irate(node_pressure_io_waiting_seconds_total{instance=\"$node\",job=\"$job\"}[$__rate_interval])",
          "format": "time_series",
          "hide": false,
          "instant": true,
          "intervalFactor": 1,
          "legendFormat": "I/O",
          "range": false,
          "refId": "I/O some",
          "step": 240
        }
      ],
      "title": "Pressure",
      "type": "bargauge"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "{{ prometheus_DS_uid }}"
      },
      "description": "Busy state of all CPU cores together",
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "thresholds"
          },
          "decimals": 1,
          "mappings": [
            {
              "options": {
                "match": "null",
                "result": {
                  "text": "N/A"
                }
              },
              "type": "special"
            }
          ],
          "max": 100,
          "min": 0,
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "rgba(50, 172, 45, 0.97)",
                "value": null
              },
              {
                "color": "rgba(237, 129, 40, 0.89)",
                "value": 85
              },
              {
                "color": "rgba(245, 54, 54, 0.9)",
                "value": 95
              }
            ]
          },
          "unit": "percent"
        },
        "overrides": []
      },
      "gridPos": {
        "h": 4,
        "w": 3,
        "x": 3,
        "y": 12
      },
      "id": 20,
      "options": {
        "minVizHeight": 75,
        "minVizWidth": 75,
        "orientation": "auto",
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "showThresholdLabels": false,
        "showThresholdMarkers": true,
        "sizing": "auto"
      },
      "pluginVersion": "10.4.1",
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "{{ prometheus_DS_uid }}"
          },
          "editorMode": "code",
          "exemplar": false,
          "expr": "100 * (1 - avg(rate(node_cpu_seconds_total{mode=\"idle\", instance=\"$node\"}[$__rate_interval])))",
          "hide": false,
          "instant": true,
          "intervalFactor": 1,
          "legendFormat": "",
          "range": false,
          "refId": "A",
          "step": 240
        }
      ],
      "title": "CPU Busy",
      "type": "gauge"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "{{ prometheus_DS_uid }}"
      },
      "description": "System load  over all CPU cores together",
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "thresholds"
          },
          "decimals": 1,
          "mappings": [
            {
              "options": {
                "match": "null",
                "result": {
                  "text": "N/A"
                }
              },
              "type": "special"
            }
          ],
          "max": 100,
          "min": 0,
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "rgba(50, 172, 45, 0.97)",
                "value": null
              },
              {
                "color": "rgba(237, 129, 40, 0.89)",
                "value": 85
              },
              {
                "color": "rgba(245, 54, 54, 0.9)",
                "value": 95
              }
            ]
          },
          "unit": "percent"
        },
        "overrides": []
      },
      "gridPos": {
        "h": 4,
        "w": 3,
        "x": 6,
        "y": 12
      },
      "id": 155,
      "options": {
        "minVizHeight": 75,
        "minVizWidth": 75,
        "orientation": "auto",
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "showThresholdLabels": false,
        "showThresholdMarkers": true,
        "sizing": "auto"
      },
      "pluginVersion": "10.4.1",
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "{{ prometheus_DS_uid }}"
          },
          "editorMode": "code",
          "exemplar": false,
          "expr": "scalar(node_load1{instance=\"$node\",job=\"$job\"}) * 100 / count(count(node_cpu_seconds_total{instance=\"$node\",job=\"$job\"}) by (cpu))",
          "format": "time_series",
          "hide": false,
          "instant": true,
          "intervalFactor": 1,
          "range": false,
          "refId": "A",
          "step": 240
        }
      ],
      "title": "Sys Load",
      "type": "gauge"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "{{ prometheus_DS_uid }}"
      },
      "description": "Non available RAM memory",
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "thresholds"
          },
          "decimals": 1,
          "mappings": [],
          "max": 100,
          "min": 0,
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "rgba(50, 172, 45, 0.97)",
                "value": null
              },
              {
                "color": "rgba(237, 129, 40, 0.89)",
                "value": 80
              },
              {
                "color": "rgba(245, 54, 54, 0.9)",
                "value": 90
              }
            ]
          },
          "unit": "percent"
        },
        "overrides": []
      },
      "gridPos": {
        "h": 4,
        "w": 3,
        "x": 9,
        "y": 12
      },
      "hideTimeOverride": false,
      "id": 16,
      "options": {
        "minVizHeight": 75,
        "minVizWidth": 75,
        "orientation": "auto",
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "showThresholdLabels": false,
        "showThresholdMarkers": true,
        "sizing": "auto"
      },
      "pluginVersion": "10.4.1",
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "{{ prometheus_DS_uid }}"
          },
          "editorMode": "code",
          "exemplar": false,
          "expr": "((node_memory_MemTotal_bytes{instance=\"$node\", job=\"$job\"} - node_memory_MemFree_bytes{instance=\"$node\", job=\"$job\"}) / node_memory_MemTotal_bytes{instance=\"$node\", job=\"$job\"}) * 100",
          "format": "time_series",
          "hide": true,
          "instant": true,
          "intervalFactor": 1,
          "range": false,
          "refId": "A",
          "step": 240
        },
        {
          "datasource": {
            "type": "prometheus",
            "uid": "{{ prometheus_DS_uid }}"
          },
          "editorMode": "code",
          "exemplar": false,
          "expr": "(1 - (node_memory_MemAvailable_bytes{instance=\"$node\", job=\"$job\"} / node_memory_MemTotal_bytes{instance=\"$node\", job=\"$job\"})) * 100",
          "format": "time_series",
          "hide": false,
          "instant": true,
          "intervalFactor": 1,
          "range": false,
          "refId": "B",
          "step": 240
        }
      ],
      "title": "RAM Used",
      "type": "gauge"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "{{ prometheus_DS_uid }}"
      },
      "description": "System uptime",
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "thresholds"
          },
          "decimals": 1,
          "mappings": [
            {
              "options": {
                "match": "null",
                "result": {
                  "text": "N/A"
                }
              },
              "type": "special"
            }
          ],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              },
              {
                "color": "red",
                "value": 80
              }
            ]
          },
          "unit": "s"
        },
        "overrides": []
      },
      "gridPos": {
        "h": 4,
        "w": 3,
        "x": 12,
        "y": 12
      },
      "hideTimeOverride": true,
      "id": 15,
      "maxDataPoints": 100,
      "options": {
        "colorMode": "none",
        "graphMode": "none",
        "justifyMode": "auto",
        "orientation": "horizontal",
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "showPercentChange": false,
        "textMode": "auto",
        "wideLayout": true
      },
      "pluginVersion": "10.4.1",
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "{{ prometheus_DS_uid }}"
          },
          "editorMode": "code",
          "exemplar": false,
          "expr": "node_time_seconds{instance=\"$node\",job=\"$job\"} - node_boot_time_seconds{instance=\"$node\",job=\"$job\"}",
          "instant": true,
          "intervalFactor": 1,
          "range": false,
          "refId": "A",
          "step": 240
        }
      ],
      "title": "Uptime",
      "type": "stat"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "{{ prometheus_DS_uid }}"
      },
      "description": "Total number of CPU cores",
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "thresholds"
          },
          "mappings": [
            {
              "options": {
                "match": "null",
                "result": {
                  "text": "N/A"
                }
              },
              "type": "special"
            }
          ],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              },
              {
                "color": "red",
                "value": 80
              }
            ]
          },
          "unit": "short"
        },
        "overrides": []
      },
      "gridPos": {
        "h": 4,
        "w": 3,
        "x": 15,
        "y": 12
      },
      "id": 14,
      "maxDataPoints": 100,
      "options": {
        "colorMode": "none",
        "graphMode": "none",
        "justifyMode": "auto",
        "orientation": "horizontal",
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "showPercentChange": false,
        "textMode": "auto",
        "wideLayout": true
      },
      "pluginVersion": "10.4.1",
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "{{ prometheus_DS_uid }}"
          },
          "editorMode": "code",
          "exemplar": false,
          "expr": "count(count(node_cpu_seconds_total{instance=\"$node\",job=\"$job\"}) by (cpu))",
          "instant": true,
          "legendFormat": "__auto",
          "range": false,
          "refId": "A"
        }
      ],
      "title": "CPU Cores",
      "type": "stat"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "{{ prometheus_DS_uid }}"
      },
      "description": "Total RootFS",
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "thresholds"
          },
          "decimals": 0,
          "mappings": [
            {
              "options": {
                "match": "null",
                "result": {
                  "text": "N/A"
                }
              },
              "type": "special"
            }
          ],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "rgba(50, 172, 45, 0.97)",
                "value": null
              },
              {
                "color": "rgba(237, 129, 40, 0.89)",
                "value": 70
              },
              {
                "color": "rgba(245, 54, 54, 0.9)",
                "value": 90
              }
            ]
          },
          "unit": "bytes"
        },
        "overrides": []
      },
      "gridPos": {
        "h": 4,
        "w": 3,
        "x": 18,
        "y": 12
      },
      "id": 23,
      "maxDataPoints": 100,
      "options": {
        "colorMode": "none",
        "graphMode": "none",
        "justifyMode": "auto",
        "orientation": "horizontal",
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "showPercentChange": false,
        "textMode": "auto",
        "wideLayout": true
      },
      "pluginVersion": "10.4.1",
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "{{ prometheus_DS_uid }}"
          },
          "editorMode": "code",
          "exemplar": false,
          "expr": "node_filesystem_size_bytes{instance=\"$node\",job=\"$job\",mountpoint=\"/\",fstype!=\"rootfs\"}",
          "format": "time_series",
          "hide": false,
          "instant": true,
          "intervalFactor": 1,
          "range": false,
          "refId": "A",
          "step": 240
        }
      ],
      "title": "RootFS Total",
      "type": "stat"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "{{ prometheus_DS_uid }}"
      },
      "description": "Total RAM",
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "thresholds"
          },
          "decimals": 0,
          "mappings": [
            {
              "options": {
                "match": "null",
                "result": {
                  "text": "N/A"
                }
              },
              "type": "special"
            }
          ],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              },
              {
                "color": "red",
                "value": 80
              }
            ]
          },
          "unit": "bytes"
        },
        "overrides": []
      },
      "gridPos": {
        "h": 4,
        "w": 3,
        "x": 21,
        "y": 12
      },
      "id": 75,
      "maxDataPoints": 100,
      "options": {
        "colorMode": "none",
        "graphMode": "none",
        "justifyMode": "auto",
        "orientation": "horizontal",
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "showPercentChange": false,
        "textMode": "auto",
        "wideLayout": true
      },
      "pluginVersion": "10.4.1",
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "{{ prometheus_DS_uid }}"
          },
          "editorMode": "code",
          "exemplar": false,
          "expr": "node_memory_MemTotal_bytes{instance=\"$node\",job=\"$job\"}",
          "instant": true,
          "intervalFactor": 1,
          "range": false,
          "refId": "A",
          "step": 240
        }
      ],
      "title": "RAM Total",
      "type": "stat"
    },
    {
      "collapsed": false,
      "datasource": {
        "type": "prometheus",
        "uid": "000000001"
      },
      "gridPos": {
        "h": 1,
        "w": 24,
        "x": 0,
        "y": 16
      },
      "id": 263,
      "panels": [],
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "000000001"
          },
          "refId": "A"
        }
      ],
      "title": "Basic CPU / Mem / Net / Disk",
      "type": "row"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "{{ prometheus_DS_uid }}"
      },
      "description": "Basic CPU info",
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "palette-classic"
          },
          "custom": {
            "axisBorderShow": false,
            "axisCenteredZero": false,
            "axisColorMode": "text",
            "axisLabel": "",
            "axisPlacement": "auto",
            "barAlignment": 0,
            "drawStyle": "line",
            "fillOpacity": 40,
            "gradientMode": "none",
            "hideFrom": {
              "legend": false,
              "tooltip": false,
              "viz": false
            },
            "insertNulls": false,
            "lineInterpolation": "smooth",
            "lineWidth": 1,
            "pointSize": 5,
            "scaleDistribution": {
              "type": "linear"
            },
            "showPoints": "never",
            "spanNulls": false,
            "stacking": {
              "group": "A",
              "mode": "percent"
            },
            "thresholdsStyle": {
              "mode": "off"
            }
          },
          "links": [],
          "mappings": [],
          "min": 0,
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              },
              {
                "color": "red",
                "value": 80
              }
            ]
          },
          "unit": "percentunit"
        },
        "overrides": [
          {
            "matcher": {
              "id": "byName",
              "options": "Busy Iowait"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "#890F02",
                  "mode": "fixed"
                }
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "Idle"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "#052B51",
                  "mode": "fixed"
                }
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "Busy Iowait"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "#890F02",
                  "mode": "fixed"
                }
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "Idle"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "#7EB26D",
                  "mode": "fixed"
                }
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "Busy System"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "#EAB839",
                  "mode": "fixed"
                }
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "Busy User"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "#0A437C",
                  "mode": "fixed"
                }
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "Busy Other"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "#6D1F62",
                  "mode": "fixed"
                }
              }
            ]
          }
        ]
      },
      "gridPos": {
        "h": 7,
        "w": 12,
        "x": 0,
        "y": 17
      },
      "id": 77,
      "options": {
        "legend": {
          "calcs": [],
          "displayMode": "list",
          "placement": "bottom",
          "showLegend": true,
          "width": 250
        },
        "tooltip": {
          "mode": "multi",
          "sort": "desc"
        }
      },
      "pluginVersion": "9.2.0",
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "{{ prometheus_DS_uid }}"
          },
          "editorMode": "code",
          "exemplar": false,
          "expr": "sum(irate(node_cpu_seconds_total{instance=\"$node\",job=\"$job\", mode=\"system\"}[$__rate_interval])) / scalar(count(count(node_cpu_seconds_total{instance=\"$node\",job=\"$job\"}) by (cpu)))",
          "format": "time_series",
          "hide": false,
          "instant": false,
          "intervalFactor": 1,
          "legendFormat": "Busy System",
          "range": true,
          "refId": "A",
          "step": 240
        },
        {
          "datasource": {
            "type": "prometheus",
            "uid": "{{ prometheus_DS_uid }}"
          },
          "editorMode": "code",
          "expr": "sum(irate(node_cpu_seconds_total{instance=\"$node\",job=\"$job\", mode=\"user\"}[$__rate_interval])) / scalar(count(count(node_cpu_seconds_total{instance=\"$node\",job=\"$job\"}) by (cpu)))",
          "format": "time_series",
          "hide": false,
          "intervalFactor": 1,
          "legendFormat": "Busy User",
          "range": true,
          "refId": "B",
          "step": 240
        },
        {
          "datasource": {
            "type": "prometheus",
            "uid": "{{ prometheus_DS_uid }}"
          },
          "editorMode": "code",
          "expr": "sum(irate(node_cpu_seconds_total{instance=\"$node\",job=\"$job\", mode=\"iowait\"}[$__rate_interval])) / scalar(count(count(node_cpu_seconds_total{instance=\"$node\",job=\"$job\"}) by (cpu)))",
          "format": "time_series",
          "intervalFactor": 1,
          "legendFormat": "Busy Iowait",
          "range": true,
          "refId": "C",
          "step": 240
        },
        {
          "datasource": {
            "type": "prometheus",
            "uid": "{{ prometheus_DS_uid }}"
          },
          "editorMode": "code",
          "expr": "sum(irate(node_cpu_seconds_total{instance=\"$node\",job=\"$job\", mode=~\".*irq\"}[$__rate_interval])) / scalar(count(count(node_cpu_seconds_total{instance=\"$node\",job=\"$job\"}) by (cpu)))",
          "format": "time_series",
          "intervalFactor": 1,
          "legendFormat": "Busy IRQs",
          "range": true,
          "refId": "D",
          "step": 240
        },
        {
          "datasource": {
            "type": "prometheus",
            "uid": "{{ prometheus_DS_uid }}"
          },
          "editorMode": "code",
          "expr": "sum(irate(node_cpu_seconds_total{instance=\"$node\",job=\"$job\",  mode!='idle',mode!='user',mode!='system',mode!='iowait',mode!='irq',mode!='softirq'}[$__rate_interval])) / scalar(count(count(node_cpu_seconds_total{instance=\"$node\",job=\"$job\"}) by (cpu)))",
          "format": "time_series",
          "intervalFactor": 1,
          "legendFormat": "Busy Other",
          "range": true,
          "refId": "E",
          "step": 240
        },
        {
          "datasource": {
            "type": "prometheus",
            "uid": "{{ prometheus_DS_uid }}"
          },
          "editorMode": "code",
          "expr": "sum(irate(node_cpu_seconds_total{instance=\"$node\",job=\"$job\", mode=\"idle\"}[$__rate_interval])) / scalar(count(count(node_cpu_seconds_total{instance=\"$node\",job=\"$job\"}) by (cpu)))",
          "format": "time_series",
          "intervalFactor": 1,
          "legendFormat": "Idle",
          "range": true,
          "refId": "F",
          "step": 240
        }
      ],
      "title": "CPU Basic",
      "type": "timeseries"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "{{ prometheus_DS_uid }}"
      },
      "description": "Basic memory usage",
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "palette-classic"
          },
          "custom": {
            "axisBorderShow": false,
            "axisCenteredZero": false,
            "axisColorMode": "text",
            "axisLabel": "",
            "axisPlacement": "auto",
            "barAlignment": 0,
            "drawStyle": "line",
            "fillOpacity": 40,
            "gradientMode": "none",
            "hideFrom": {
              "legend": false,
              "tooltip": false,
              "viz": false
            },
            "insertNulls": false,
            "lineInterpolation": "linear",
            "lineWidth": 1,
            "pointSize": 5,
            "scaleDistribution": {
              "type": "linear"
            },
            "showPoints": "never",
            "spanNulls": false,
            "stacking": {
              "group": "A",
              "mode": "normal"
            },
            "thresholdsStyle": {
              "mode": "off"
            }
          },
          "links": [],
          "mappings": [],
          "min": 0,
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              },
              {
                "color": "red",
                "value": 80
              }
            ]
          },
          "unit": "bytes"
        },
        "overrides": [
          {
            "matcher": {
              "id": "byName",
              "options": "Apps"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "#629E51",
                  "mode": "fixed"
                }
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "Buffers"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "#614D93",
                  "mode": "fixed"
                }
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "Cache"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "#6D1F62",
                  "mode": "fixed"
                }
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "Cached"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "#511749",
                  "mode": "fixed"
                }
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "Committed"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "#508642",
                  "mode": "fixed"
                }
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "Free"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "#0A437C",
                  "mode": "fixed"
                }
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "Hardware Corrupted - Amount of RAM that the kernel identified as corrupted / not working"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "#CFFAFF",
                  "mode": "fixed"
                }
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "Inactive"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "#584477",
                  "mode": "fixed"
                }
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "PageTables"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "#0A50A1",
                  "mode": "fixed"
                }
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "Page_Tables"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "#0A50A1",
                  "mode": "fixed"
                }
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "RAM_Free"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "#E0F9D7",
                  "mode": "fixed"
                }
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "SWAP Used"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "#BF1B00",
                  "mode": "fixed"
                }
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "Slab"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "#806EB7",
                  "mode": "fixed"
                }
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "Slab_Cache"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "#E0752D",
                  "mode": "fixed"
                }
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "Swap"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "#BF1B00",
                  "mode": "fixed"
                }
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "Swap Used"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "#BF1B00",
                  "mode": "fixed"
                }
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "Swap_Cache"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "#C15C17",
                  "mode": "fixed"
                }
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "Swap_Free"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "#2F575E",
                  "mode": "fixed"
                }
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "Unused"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "#EAB839",
                  "mode": "fixed"
                }
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "RAM Total"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "#E0F9D7",
                  "mode": "fixed"
                }
              },
              {
                "id": "custom.fillOpacity",
                "value": 0
              },
              {
                "id": "custom.stacking",
                "value": {
                  "group": false,
                  "mode": "normal"
                }
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "RAM Cache + Buffer"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "#052B51",
                  "mode": "fixed"
                }
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "RAM Free"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "#7EB26D",
                  "mode": "fixed"
                }
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "Available"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "#DEDAF7",
                  "mode": "fixed"
                }
              },
              {
                "id": "custom.fillOpacity",
                "value": 0
              },
              {
                "id": "custom.stacking",
                "value": {
                  "group": false,
                  "mode": "normal"
                }
              }
            ]
          }
        ]
      },
      "gridPos": {
        "h": 7,
        "w": 12,
        "x": 12,
        "y": 17
      },
      "id": 78,
      "options": {
        "legend": {
          "calcs": [],
          "displayMode": "list",
          "placement": "bottom",
          "showLegend": true,
          "width": 350
        },
        "tooltip": {
          "mode": "multi",
          "sort": "none"
        }
      },
      "pluginVersion": "9.2.0",
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "{{ prometheus_DS_uid }}"
          },
          "expr": "node_memory_MemTotal_bytes{instance=\"$node\",job=\"$job\"}",
          "format": "time_series",
          "hide": false,
          "intervalFactor": 1,
          "legendFormat": "RAM Total",
          "refId": "A",
          "step": 240
        },
        {
          "datasource": {
            "type": "prometheus",
            "uid": "{{ prometheus_DS_uid }}"
          },
          "expr": "node_memory_MemTotal_bytes{instance=\"$node\",job=\"$job\"} - node_memory_MemFree_bytes{instance=\"$node\",job=\"$job\"} - (node_memory_Cached_bytes{instance=\"$node\",job=\"$job\"} + node_memory_Buffers_bytes{instance=\"$node\",job=\"$job\"} + node_memory_SReclaimable_bytes{instance=\"$node\",job=\"$job\"})",
          "format": "time_series",
          "hide": false,
          "intervalFactor": 1,
          "legendFormat": "RAM Used",
          "refId": "B",
          "step": 240
        },
        {
          "datasource": {
            "type": "prometheus",
            "uid": "{{ prometheus_DS_uid }}"
          },
          "expr": "node_memory_Cached_bytes{instance=\"$node\",job=\"$job\"} + node_memory_Buffers_bytes{instance=\"$node\",job=\"$job\"} + node_memory_SReclaimable_bytes{instance=\"$node\",job=\"$job\"}",
          "format": "time_series",
          "intervalFactor": 1,
          "legendFormat": "RAM Cache + Buffer",
          "refId": "C",
          "step": 240
        },
        {
          "datasource": {
            "type": "prometheus",
            "uid": "{{ prometheus_DS_uid }}"
          },
          "expr": "node_memory_MemFree_bytes{instance=\"$node\",job=\"$job\"}",
          "format": "time_series",
          "intervalFactor": 1,
          "legendFormat": "RAM Free",
          "refId": "D",
          "step": 240
        },
        {
          "datasource": {
            "type": "prometheus",
            "uid": "{{ prometheus_DS_uid }}"
          },
          "expr": "(node_memory_SwapTotal_bytes{instance=\"$node\",job=\"$job\"} - node_memory_SwapFree_bytes{instance=\"$node\",job=\"$job\"})",
          "format": "time_series",
          "intervalFactor": 1,
          "legendFormat": "SWAP Used",
          "refId": "E",
          "step": 240
        }
      ],
      "title": "Memory Basic",
      "type": "timeseries"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "{{ prometheus_DS_uid }}"
      },
      "description": "Basic network info per interface",
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "palette-classic"
          },
          "custom": {
            "axisBorderShow": false,
            "axisCenteredZero": false,
            "axisColorMode": "text",
            "axisLabel": "",
            "axisPlacement": "auto",
            "barAlignment": 0,
            "drawStyle": "line",
            "fillOpacity": 40,
            "gradientMode": "none",
            "hideFrom": {
              "legend": false,
              "tooltip": false,
              "viz": false
            },
            "insertNulls": false,
            "lineInterpolation": "linear",
            "lineWidth": 1,
            "pointSize": 5,
            "scaleDistribution": {
              "type": "linear"
            },
            "showPoints": "never",
            "spanNulls": false,
            "stacking": {
              "group": "A",
              "mode": "none"
            },
            "thresholdsStyle": {
              "mode": "off"
            }
          },
          "links": [],
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              },
              {
                "color": "red",
                "value": 80
              }
            ]
          },
          "unit": "bps"
        },
        "overrides": [
          {
            "matcher": {
              "id": "byName",
              "options": "Recv_bytes_eth2"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "#7EB26D",
                  "mode": "fixed"
                }
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "Recv_bytes_lo"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "#0A50A1",
                  "mode": "fixed"
                }
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "Recv_drop_eth2"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "#6ED0E0",
                  "mode": "fixed"
                }
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "Recv_drop_lo"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "#E0F9D7",
                  "mode": "fixed"
                }
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "Recv_errs_eth2"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "#BF1B00",
                  "mode": "fixed"
                }
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "Recv_errs_lo"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "#CCA300",
                  "mode": "fixed"
                }
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "Trans_bytes_eth2"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "#7EB26D",
                  "mode": "fixed"
                }
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "Trans_bytes_lo"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "#0A50A1",
                  "mode": "fixed"
                }
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "Trans_drop_eth2"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "#6ED0E0",
                  "mode": "fixed"
                }
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "Trans_drop_lo"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "#E0F9D7",
                  "mode": "fixed"
                }
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "Trans_errs_eth2"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "#BF1B00",
                  "mode": "fixed"
                }
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "Trans_errs_lo"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "#CCA300",
                  "mode": "fixed"
                }
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "recv_bytes_lo"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "#0A50A1",
                  "mode": "fixed"
                }
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "recv_drop_eth0"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "#99440A",
                  "mode": "fixed"
                }
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "recv_drop_lo"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "#967302",
                  "mode": "fixed"
                }
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "recv_errs_eth0"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "#BF1B00",
                  "mode": "fixed"
                }
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "recv_errs_lo"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "#890F02",
                  "mode": "fixed"
                }
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "trans_bytes_eth0"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "#7EB26D",
                  "mode": "fixed"
                }
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "trans_bytes_lo"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "#0A50A1",
                  "mode": "fixed"
                }
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "trans_drop_eth0"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "#99440A",
                  "mode": "fixed"
                }
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "trans_drop_lo"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "#967302",
                  "mode": "fixed"
                }
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "trans_errs_eth0"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "#BF1B00",
                  "mode": "fixed"
                }
              }
            ]
          },
          {
            "matcher": {
              "id": "byName",
              "options": "trans_errs_lo"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "#890F02",
                  "mode": "fixed"
                }
              }
            ]
          },
          {
            "matcher": {
              "id": "byRegexp",
              "options": "/.*trans.*/"
            },
            "properties": [
              {
                "id": "custom.transform",
                "value": "negative-Y"
              }
            ]
          }
        ]
      },
      "gridPos": {
        "h": 7,
        "w": 12,
        "x": 0,
        "y": 24
      },
      "id": 74,
      "options": {
        "legend": {
          "calcs": [],
          "displayMode": "list",
          "placement": "bottom",
          "showLegend": true
        },
        "tooltip": {
          "mode": "multi",
          "sort": "none"
        }
      },
      "pluginVersion": "9.2.0",
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "{{ prometheus_DS_uid }}"
          },
          "expr": "irate(node_network_receive_bytes_total{instance=\"$node\",job=\"$job\"}[$__rate_interval])*8",
          "format": "time_series",
          "intervalFactor": 1,
          "legendFormat": "recv {% raw %}{{device}}{% endraw %}",
          "refId": "A",
          "step": 240
        },
        {
          "datasource": {
            "type": "prometheus",
            "uid": "{{ prometheus_DS_uid }}"
          },
          "expr": "irate(node_network_transmit_bytes_total{instance=\"$node\",job=\"$job\"}[$__rate_interval])*8",
          "format": "time_series",
          "intervalFactor": 1,
          "legendFormat": "trans {% raw %}{{device}}{% endraw %}",
          "refId": "B",
          "step": 240
        }
      ],
      "title": "Network Traffic Basic",
      "type": "timeseries"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "{{ prometheus_DS_uid }}"
      },
      "description": "Disk space used of all filesystems mounted",
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "palette-classic"
          },
          "custom": {
            "axisBorderShow": false,
            "axisCenteredZero": false,
            "axisColorMode": "text",
            "axisLabel": "",
            "axisPlacement": "auto",
            "barAlignment": 0,
            "drawStyle": "line",
            "fillOpacity": 40,
            "gradientMode": "none",
            "hideFrom": {
              "legend": false,
              "tooltip": false,
              "viz": false
            },
            "insertNulls": false,
            "lineInterpolation": "linear",
            "lineWidth": 1,
            "pointSize": 5,
            "scaleDistribution": {
              "type": "linear"
            },
            "showPoints": "never",
            "spanNulls": false,
            "stacking": {
              "group": "A",
              "mode": "none"
            },
            "thresholdsStyle": {
              "mode": "off"
            }
          },
          "links": [],
          "mappings": [],
          "max": 100,
          "min": 0,
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              },
              {
                "color": "red",
                "value": 80
              }
            ]
          },
          "unit": "percent"
        },
        "overrides": []
      },
      "gridPos": {
        "h": 7,
        "w": 12,
        "x": 12,
        "y": 24
      },
      "id": 152,
      "options": {
        "legend": {
          "calcs": [],
          "displayMode": "list",
          "placement": "bottom",
          "showLegend": true
        },
        "tooltip": {
          "mode": "multi",
          "sort": "none"
        }
      },
      "pluginVersion": "9.2.0",
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "{{ prometheus_DS_uid }}"
          },
          "expr": "100 - ((node_filesystem_avail_bytes{instance=\"$node\",job=\"$job\",device!~'rootfs'} * 100) / node_filesystem_size_bytes{instance=\"$node\",job=\"$job\",device!~'rootfs'})",
          "format": "time_series",
          "intervalFactor": 1,
          "legendFormat": "{% raw %}{{mountpoint}}{% endraw %}",
          "refId": "A",
          "step": 240
        }
      ],
      "title": "Disk Space Used Basic",
      "type": "timeseries"
    }
  ],
  "refresh": "10s",
  "revision": 1,
  "schemaVersion": 39,
  "tags": [
    "linux"
  ],
  "templating": {
    "list": [
      {
        "current": {
          "selected": true,
          "text": "p7-prometheus",
          "value": "{{ prometheus_DS_uid }}"
        },
        "hide": 0,
        "includeAll": false,
        "label": "Datasource",
        "multi": false,
        "name": "datasource",
        "options": [],
        "query": "prometheus",
        "queryValue": "",
        "refresh": 1,
        "regex": "",
        "skipUrlSync": false,
        "type": "datasource"
      },
      {
        "current": {},
        "datasource": {
          "type": "prometheus",
          "uid": "{{ prometheus_DS_uid }}"
        },
        "definition": "",
        "hide": 0,
        "includeAll": false,
        "label": "Job",
        "multi": false,
        "name": "job",
        "options": [],
        "query": {
          "query": "label_values(node_uname_info, job)",
          "refId": "Prometheus-job-Variable-Query"
        },
        "refresh": 1,
        "regex": "",
        "skipUrlSync": false,
        "sort": 1,
        "tagValuesQuery": "",
        "tagsQuery": "",
        "type": "query",
        "useTags": false
      },
      {
        "current": {},
        "datasource": {
          "type": "prometheus",
          "uid": "{{ prometheus_DS_uid }}"
        },
        "definition": "label_values(node_uname_info{job=\"$job\"}, instance)",
        "hide": 0,
        "includeAll": false,
        "label": "Host",
        "multi": false,
        "name": "node",
        "options": [],
        "query": {
          "query": "label_values(node_uname_info{job=\"$job\"}, instance)",
          "refId": "Prometheus-node-Variable-Query"
        },
        "refresh": 1,
        "regex": "",
        "skipUrlSync": false,
        "sort": 1,
        "tagValuesQuery": "",
        "tagsQuery": "",
        "type": "query",
        "useTags": false
      },
      {
        "current": {
          "selected": false,
          "text": "[a-z]+|nvme[0-9]+n[0-9]+|mmcblk[0-9]+",
          "value": "[a-z]+|nvme[0-9]+n[0-9]+|mmcblk[0-9]+"
        },
        "hide": 2,
        "includeAll": false,
        "multi": false,
        "name": "diskdevices",
        "options": [
          {
            "selected": true,
            "text": "[a-z]+|nvme[0-9]+n[0-9]+|mmcblk[0-9]+",
            "value": "[a-z]+|nvme[0-9]+n[0-9]+|mmcblk[0-9]+"
          }
        ],
        "query": "[a-z]+|nvme[0-9]+n[0-9]+|mmcblk[0-9]+",
        "skipUrlSync": false,
        "type": "custom"
      }
    ]
  },
  "time": {
    "from": "now-5m",
    "to": "now"
  },
  "timepicker": {
    "refresh_intervals": [
      "5s",
      "10s",
      "30s",
      "1m",
      "5m",
      "15m",
      "30m",
      "1h",
      "2h",
      "1d"
    ],
    "time_options": [
      "5m",
      "15m",
      "1h",
      "6h",
      "12h",
      "24h",
      "2d",
      "7d",
      "30d"
    ]
  },
  "timezone": "browser",
  "title": "{{ dashboard_name }}",
  "uid": "{{ dashboard_uid }}",
  "version": 8,
  "weekStart": ""
}
```
</details>

***

### tests file (test.yml)

This Ansible playbook named "test.yml" targets the localhost machine and utilizes the "Attendance Grafana Dashboard" role to add dashboard  in PGA Server.

<details>
<summary> Click here to see test.yml file</summary>
<br>
  
```shell
---
- hosts: localhost
  become: yes
  gather_facts: yes
  roles:
    - attendanceGfDashboard

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
# vars file for attendanceGfDashboard
editor_name : "Harshit"
grafana_api_key: "glsa_oIJNxnxI1QAl6alNgCPtF8SgkEXURjpc_8c78e79d"
grafana_url: http://18.207.163.171:3000
json_file_template_src: "attendanceDashboard.json.j2"
json_file_dest: "/home/ubuntu/attendanceDashboard.json"

#  template
prometheus_DS_uid : "fdi6x7lbmqxvkf"
dashboard_name: "Attendance-API"
dashboard_uid: "007"
```
</details>

***

# Output

### Role Execution

***

### PGA Server 

***

### PGA Server Dashboard View

***

# Conclusion

This guide illustrates the process of adding **Attendance Grafana Dashboard** through Ansible. By adhering to these instructions, you can effectively visualize and monitor your `Attendance APi Server` within your AWS infrastructure.

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
| Ansible Module for Grafana Dashboard| [Reference Link](https://docs.ansible.com/ansible/latest/collections/community/grafana/grafana_dashboard_module.html)|
| Grafana Provisioning | [Reference Link](https://grafana.com/docs/grafana/latest/administration/provisioning/) |
