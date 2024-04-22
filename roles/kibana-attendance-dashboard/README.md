# Ansible Role for Attendance Kibana Dashboard

![image](https://github.com/CodeOps-Hub/Ansible/assets/156056349/f87d9c4a-4500-47e2-a452-e6c8192dabc1)

***
|   Author        |  Created on   |  Version   | Last updated by  | Last edited on |
| --------------- | --------------| -----------|----------------- | -------------- |
| **Vidhi Yadav**    | 20 April 2024 |  Version 1 | Vidhi Yadav    | 20 April 2024  |
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

This document is created to streamline the automation of adding the Attendance Kibana Dashboard to the target server. Its purpose is to simplify the process and provide clear explanations of the role created and utilized for the setup of the Attendance Kibana Dashboard.

***

# Flow Diagram

<img width="1270" alt="Screenshot 2024-04-20 at 4 15 07 AM" src="https://github.com/CodeOps-Hub/Ansible/assets/156056349/b6111af8-5919-492e-8c6b-33ef0b5bdd06">


***

# Pre-requisites

| **Pre-requisite** | **Description** |
| ----------------- | --------------- |
| **Ansible**       | Ansible must be installed on the control machine from which you plan to run the playbook. |
| **AWS CLI**       | AWS CLI for providing aws credentials to fetch resources. |
| **Python**        | Ensure that Python is installed on your system. Ansible relies on Python for its execution, and dynamic inventory scripts are typically written in Python. |
| **PIP (Python Package Installer)** | Install pip if it's not already installed. pip is a package manager for Python that allows you to install and manage Python packages. |
| **Boto3**   |  If your dynamic inventory script relies on AWS APIs to fetch inventory data, you'll need to install `boto3` using `pip`. |
| **EFK Server** | Must have installed `Elasticsearch, fluentd and Kibana` having necessary ports allowed on it. |

***

# Directory Structure

<img width="598" alt="Screenshot 2024-04-20 at 5 35 43 AM" src="https://github.com/CodeOps-Hub/Ansible/assets/156056349/f81200aa-949d-4ca0-a39a-a9ed29c48fdb">

***

> [!NOTE]
>Ensure that for dynamic inventory you have the necessary AWS credentials configured in AWS CLI or an IAM role on the node.

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
private_key_file = /path/to/private-key

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

### kibana_dashboard.yml file

<details>
<summary> Click here to see playbook.yml file</summary>
<br>
  
```shell
---
- hosts: efk
  become: yes
  gather_facts: yes
  roles:
    - kibana-role
```
</details>

***

## Attendance Kibana Dashboard Role

### addDashboard file (addDashboard.yml)
This Ansible playbook consists of tasks aimed at copying json file , adding dashboard in Kibana then deleting that json file.
<details>
<summary> Click here to see ubuntu.yml file</summary>
<br>
  
```shell
---

- name: Import json file template
  ansible.builtin.template:
    src: "{{ json_file_template_src }}"
    dest: "{{ json_file_dest }}"

- name: Import Kibana dashboard
  shell: "curl -k -X POST {{ kibana_dashboard_import_url }} -H 'kbn-xsrf: true' --form file=@{{ json_file_dest }}"

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
# tasks file for attendance Kibana Dashboard
- name: Include tasks for Attendance 
  ansible.builtin.include_tasks: addDashboard.yml

```
</details>

***

### templates file (Kibana-attendance-API.ndjson.j2)


<details>
<summary> Click here to see  file</summary>
<br>
  
```shell
{"attributes":{"allowHidden":false,"fieldAttrs":"{}","fieldFormatMap":"{}","fields":"[]","name":"API logs","runtimeFieldMap":"{}","sourceFilters":"[]","timeFieldName":"@timestamp","title":"logs*"},"coreMigrationVersion":"8.8.0","created_at":"2024-04-19T10:38:29.486Z","id":"0ed79425-3edd-4496-9565-6eb60db97434","managed":false,"references":[],"type":"index-pattern","typeMigrationVersion":"8.0.0","updated_at":"2024-04-19T16:31:00.809Z","version":"WzE3NywyXQ=="}
{"attributes":{"description":"","kibanaSavedObjectMeta":{"searchSourceJSON":"{\"query\":{\"query\":\"\",\"language\":\"kuery\"},\"filter\":[],\"indexRefName\":\"kibanaSavedObjectMeta.searchSourceJSON.index\"}"},"title":"status code","uiStateJSON":"{}","version":1,"visState":"{\"title\":\"status code\",\"type\":\"pie\",\"aggs\":[{\"id\":\"1\",\"enabled\":true,\"type\":\"count\",\"params\":{\"emptyAsNull\":false},\"schema\":\"metric\"},{\"id\":\"2\",\"enabled\":true,\"type\":\"terms\",\"params\":{\"field\":\"status_code.keyword\",\"orderBy\":\"custom\",\"orderAgg\":{\"id\":\"2-orderAgg\",\"enabled\":true,\"type\":\"count\",\"params\":{\"emptyAsNull\":false},\"schema\":\"orderAgg\"},\"order\":\"desc\",\"size\":5,\"otherBucket\":false,\"otherBucketLabel\":\"Other\",\"missingBucket\":false,\"missingBucketLabel\":\"Missing\",\"includeIsRegex\":true,\"excludeIsRegex\":true,\"customLabel\":\"Attendance API\"},\"schema\":\"segment\"}],\"params\":{\"type\":\"pie\",\"addTooltip\":true,\"legendDisplay\":\"hide\",\"legendPosition\":\"right\",\"nestedLegend\":false,\"truncateLegend\":true,\"maxLegendLines\":1,\"distinctColors\":false,\"isDonut\":true,\"emptySizeRatio\":0.3,\"palette\":{\"type\":\"palette\",\"name\":\"default\"},\"labels\":{\"show\":true,\"last_level\":false,\"values\":true,\"valuesFormat\":\"percent\",\"percentDecimals\":2,\"truncate\":100,\"position\":\"default\"}}}"},"coreMigrationVersion":"8.8.0","created_at":"2024-04-19T10:55:00.252Z","id":"cd1cbacf-d2e0-4eff-8dc2-97c478198e70","managed":false,"references":[{"id":"0ed79425-3edd-4496-9565-6eb60db97434","name":"kibanaSavedObjectMeta.searchSourceJSON.index","type":"index-pattern"}],"type":"visualization","typeMigrationVersion":"8.5.0","updated_at":"2024-04-19T10:55:00.252Z","version":"Wzc0LDJd"}
{"attributes":{"description":"","kibanaSavedObjectMeta":{"searchSourceJSON":"{\"query\":{\"query\":\"\",\"language\":\"kuery\"},\"filter\":[]}"},"optionsJSON":"{\"useMargins\":true,\"syncColors\":false,\"syncCursor\":true,\"syncTooltips\":false,\"hidePanelTitles\":false}","panelsJSON":"[{\"type\":\"lens\",\"gridData\":{\"x\":0,\"y\":0,\"w\":21,\"h\":13,\"i\":\"06b54e54-4396-4a1e-a461-92a016b73cc8\"},\"panelIndex\":\"06b54e54-4396-4a1e-a461-92a016b73cc8\",\"embeddableConfig\":{\"attributes\":{\"title\":\"\",\"visualizationType\":\"lnsXY\",\"type\":\"lens\",\"references\":[{\"type\":\"index-pattern\",\"id\":\"0ed79425-3edd-4496-9565-6eb60db97434\",\"name\":\"indexpattern-datasource-layer-5e296f9c-d16f-4861-9a5c-16687ce65c47\"}],\"state\":{\"visualization\":{\"legend\":{\"isVisible\":true,\"position\":\"right\"},\"valueLabels\":\"hide\",\"fittingFunction\":\"None\",\"axisTitlesVisibilitySettings\":{\"x\":true,\"yLeft\":true,\"yRight\":true},\"tickLabelsVisibilitySettings\":{\"x\":true,\"yLeft\":true,\"yRight\":true},\"labelsOrientation\":{\"x\":0,\"yLeft\":0,\"yRight\":0},\"gridlinesVisibilitySettings\":{\"x\":true,\"yLeft\":true,\"yRight\":true},\"preferredSeriesType\":\"bar_stacked\",\"layers\":[{\"layerId\":\"5e296f9c-d16f-4861-9a5c-16687ce65c47\",\"layerType\":\"data\",\"accessors\":[\"157ffd66-c714-429b-a113-be4aa24c97c6\"],\"seriesType\":\"bar_stacked\",\"xAccessor\":\"f87ce69e-1f41-4875-9551-f55948eac39a\",\"colorMapping\":{\"assignments\":[],\"specialAssignments\":[{\"rule\":{\"type\":\"other\"},\"color\":{\"type\":\"loop\"},\"touched\":false}],\"paletteId\":\"eui_amsterdam_color_blind\",\"colorMode\":{\"type\":\"categorical\"}}}]},\"query\":{\"query\":\"\",\"language\":\"kuery\"},\"filters\":[],\"datasourceStates\":{\"formBased\":{\"layers\":{\"5e296f9c-d16f-4861-9a5c-16687ce65c47\":{\"columns\":{\"f87ce69e-1f41-4875-9551-f55948eac39a\":{\"label\":\"Filters\",\"dataType\":\"string\",\"operationType\":\"filters\",\"scale\":\"ordinal\",\"isBucketed\":true,\"params\":{\"filters\":[{\"label\":\"Database Health\",\"input\":{\"query\":\"\\\"request.keyword\\\" :\\\"GET /api/v1/attendance/health/detail HTTP/1.1\\\" and status_code : 200\",\"language\":\"kuery\"}},{\"input\":{\"query\":\"request.keyword :\\\"GET /api/v1/attendance/health HTTP/1.1\\\" and status_code : 200\",\"language\":\"kuery\"},\"label\":\"API Health\"}]}},\"157ffd66-c714-429b-a113-be4aa24c97c6\":{\"label\":\"Count of records\",\"dataType\":\"number\",\"operationType\":\"count\",\"isBucketed\":false,\"scale\":\"ratio\",\"sourceField\":\"___records___\",\"params\":{\"emptyAsNull\":true}}},\"columnOrder\":[\"f87ce69e-1f41-4875-9551-f55948eac39a\",\"157ffd66-c714-429b-a113-be4aa24c97c6\"],\"incompleteColumns\":{},\"sampling\":1}}},\"indexpattern\":{\"layers\":{}},\"textBased\":{\"layers\":{}}},\"internalReferences\":[],\"adHocDataViews\":{}}},\"enhancements\":{}},\"title\":\"Health\"},{\"type\":\"visualization\",\"gridData\":{\"x\":21,\"y\":0,\"w\":20,\"h\":13,\"i\":\"3b1c8f1e-1fd4-4bb0-a342-3ec35ce30249\"},\"panelIndex\":\"3b1c8f1e-1fd4-4bb0-a342-3ec35ce30249\",\"embeddableConfig\":{\"enhancements\":{}},\"title\":\"Status codes\",\"panelRefName\":\"panel_3b1c8f1e-1fd4-4bb0-a342-3ec35ce30249\"},{\"type\":\"lens\",\"gridData\":{\"x\":0,\"y\":13,\"w\":24,\"h\":15,\"i\":\"82838041-2c93-4121-b77e-e88441852351\"},\"panelIndex\":\"82838041-2c93-4121-b77e-e88441852351\",\"embeddableConfig\":{\"attributes\":{\"title\":\"\",\"visualizationType\":\"lnsXY\",\"type\":\"lens\",\"references\":[{\"type\":\"index-pattern\",\"id\":\"0ed79425-3edd-4496-9565-6eb60db97434\",\"name\":\"indexpattern-datasource-layer-53c4bb90-1645-4bd5-8051-af52a9503657\"}],\"state\":{\"visualization\":{\"legend\":{\"isVisible\":true,\"position\":\"right\"},\"valueLabels\":\"hide\",\"fittingFunction\":\"None\",\"axisTitlesVisibilitySettings\":{\"x\":true,\"yLeft\":true,\"yRight\":true},\"tickLabelsVisibilitySettings\":{\"x\":true,\"yLeft\":true,\"yRight\":true},\"labelsOrientation\":{\"x\":0,\"yLeft\":0,\"yRight\":0},\"gridlinesVisibilitySettings\":{\"x\":true,\"yLeft\":true,\"yRight\":true},\"preferredSeriesType\":\"bar\",\"layers\":[{\"layerId\":\"53c4bb90-1645-4bd5-8051-af52a9503657\",\"accessors\":[\"231c4f3d-69d3-43b7-8a51-02eb035e46a1\"],\"position\":\"top\",\"seriesType\":\"bar\",\"showGridlines\":false,\"layerType\":\"data\",\"colorMapping\":{\"assignments\":[],\"specialAssignments\":[{\"rule\":{\"type\":\"other\"},\"color\":{\"type\":\"loop\"},\"touched\":false}],\"paletteId\":\"eui_amsterdam_color_blind\",\"colorMode\":{\"type\":\"categorical\"}},\"xAccessor\":\"c001d27a-31d8-43d0-b7cd-76f6d0463317\",\"yConfig\":[{\"forAccessor\":\"231c4f3d-69d3-43b7-8a51-02eb035e46a1\",\"color\":\"#b9a888\",\"axisMode\":\"auto\"}]}]},\"query\":{\"query\":\"\",\"language\":\"kuery\"},\"filters\":[{\"meta\":{\"index\":\"e192c222-de6e-41f5-89fd-30629683964a\",\"alias\":\"Method\",\"type\":\"custom\",\"key\":\"query\",\"value\":\"{\\\"bool\\\":{\\\"must\\\":[],\\\"filter\\\":[{\\\"bool\\\":{\\\"should\\\":[{\\\"exists\\\":{\\\"field\\\":\\\"method\\\"}}],\\\"minimum_should_match\\\":1}}],\\\"should\\\":[],\\\"must_not\\\":[]}}\",\"disabled\":false,\"negate\":false},\"query\":{\"bool\":{\"must\":[],\"filter\":[{\"bool\":{\"should\":[{\"exists\":{\"field\":\"method\"}}],\"minimum_should_match\":1}}],\"should\":[],\"must_not\":[]}},\"$state\":{\"store\":\"appState\"}}],\"datasourceStates\":{\"formBased\":{\"layers\":{\"53c4bb90-1645-4bd5-8051-af52a9503657\":{\"columns\":{\"c001d27a-31d8-43d0-b7cd-76f6d0463317\":{\"label\":\"Method\",\"dataType\":\"string\",\"operationType\":\"filters\",\"scale\":\"ordinal\",\"isBucketed\":true,\"params\":{\"filters\":[{\"input\":{\"query\":\"method : POST\",\"language\":\"kuery\"},\"label\":\"POST\"},{\"input\":{\"query\":\"method : GET\",\"language\":\"kuery\"},\"label\":\"GET\"}]},\"customLabel\":true},\"231c4f3d-69d3-43b7-8a51-02eb035e46a1\":{\"label\":\"Hourly Basis\",\"dataType\":\"number\",\"operationType\":\"count\",\"isBucketed\":false,\"scale\":\"ratio\",\"sourceField\":\"method.keyword\",\"timeScale\":\"h\",\"filter\":{\"query\":\"method : *\",\"language\":\"kuery\"},\"timeShift\":\"\",\"params\":{\"emptyAsNull\":true},\"customLabel\":true}},\"columnOrder\":[\"c001d27a-31d8-43d0-b7cd-76f6d0463317\",\"231c4f3d-69d3-43b7-8a51-02eb035e46a1\"],\"sampling\":1,\"ignoreGlobalFilters\":false,\"incompleteColumns\":{},\"indexPatternId\":\"0ed79425-3edd-4496-9565-6eb60db97434\"}},\"currentIndexPatternId\":\"0ed79425-3edd-4496-9565-6eb60db97434\"},\"indexpattern\":{\"layers\":{}},\"textBased\":{\"layers\":{}}},\"internalReferences\":[],\"adHocDataViews\":{}}},\"enhancements\":{}},\"title\":\"HTTP method\"},{\"type\":\"lens\",\"gridData\":{\"x\":24,\"y\":13,\"w\":21,\"h\":13,\"i\":\"6c8f2103-5131-4f19-9d06-2921d98fb7fe\"},\"panelIndex\":\"6c8f2103-5131-4f19-9d06-2921d98fb7fe\",\"embeddableConfig\":{\"attributes\":{\"title\":\"\",\"visualizationType\":\"lnsXY\",\"type\":\"lens\",\"references\":[{\"type\":\"index-pattern\",\"id\":\"0ed79425-3edd-4496-9565-6eb60db97434\",\"name\":\"indexpattern-datasource-layer-81e66199-62be-4aec-ac23-4caf81af88c5\"}],\"state\":{\"visualization\":{\"legend\":{\"isVisible\":true,\"position\":\"right\"},\"valueLabels\":\"hide\",\"fittingFunction\":\"None\",\"axisTitlesVisibilitySettings\":{\"x\":true,\"yLeft\":true,\"yRight\":true},\"tickLabelsVisibilitySettings\":{\"x\":true,\"yLeft\":true,\"yRight\":true},\"labelsOrientation\":{\"x\":0,\"yLeft\":0,\"yRight\":0},\"gridlinesVisibilitySettings\":{\"x\":true,\"yLeft\":true,\"yRight\":true},\"preferredSeriesType\":\"bar_horizontal\",\"layers\":[{\"layerId\":\"81e66199-62be-4aec-ac23-4caf81af88c5\",\"seriesType\":\"bar_horizontal\",\"xAccessor\":\"b5d89b8c-361a-4e52-b898-efa44c7ece4a\",\"splitAccessor\":\"9a9be21e-5cbd-445f-aa48-38cca5e87a16\",\"accessors\":[\"a722f9a5-a882-41f2-b53a-0726fb2d1665\"],\"layerType\":\"data\",\"colorMapping\":{\"assignments\":[{\"rule\":{\"type\":\"matchExactly\",\"values\":[\"Error Codes\",\"500\"]},\"color\":{\"type\":\"categorical\",\"paletteId\":\"eui_amsterdam_color_blind\",\"colorIndex\":9},\"touched\":true},{\"rule\":{\"type\":\"auto\"},\"color\":{\"type\":\"colorCode\",\"colorCode\":\"#e8a27f\"},\"touched\":true}],\"specialAssignments\":[{\"rule\":{\"type\":\"other\"},\"color\":{\"type\":\"loop\"},\"touched\":true}],\"paletteId\":\"eui_amsterdam_color_blind\",\"colorMode\":{\"type\":\"categorical\"}}}]},\"query\":{\"query\":\"\",\"language\":\"kuery\"},\"filters\":[],\"datasourceStates\":{\"formBased\":{\"layers\":{\"81e66199-62be-4aec-ac23-4caf81af88c5\":{\"columns\":{\"b5d89b8c-361a-4e52-b898-efa44c7ece4a\":{\"label\":\"Timestamp\",\"dataType\":\"date\",\"operationType\":\"date_histogram\",\"sourceField\":\"@timestamp\",\"isBucketed\":true,\"scale\":\"interval\",\"params\":{\"interval\":\"h\",\"includeEmptyRows\":true,\"dropPartials\":false},\"customLabel\":true},\"9a9be21e-5cbd-445f-aa48-38cca5e87a16\":{\"label\":\"Filters\",\"dataType\":\"string\",\"operationType\":\"filters\",\"scale\":\"ordinal\",\"isBucketed\":true,\"params\":{\"filters\":[{\"label\":\"Error Codes\",\"input\":{\"query\":\"status_code : 500 or status_code : 404\",\"language\":\"kuery\"}},{\"input\":{\"query\":\"status_code : 500 or status_code : 404\",\"language\":\"kuery\"},\"label\":\"\"}]}},\"a722f9a5-a882-41f2-b53a-0726fb2d1665\":{\"label\":\"Error Code Count\",\"dataType\":\"number\",\"operationType\":\"count\",\"isBucketed\":false,\"scale\":\"ratio\",\"sourceField\":\"status_code.keyword\",\"filter\":{\"query\":\"status_code : 500 or status_code : 404\",\"language\":\"kuery\"},\"params\":{\"emptyAsNull\":true},\"customLabel\":true}},\"columnOrder\":[\"b5d89b8c-361a-4e52-b898-efa44c7ece4a\",\"9a9be21e-5cbd-445f-aa48-38cca5e87a16\",\"a722f9a5-a882-41f2-b53a-0726fb2d1665\"],\"incompleteColumns\":{},\"sampling\":1,\"indexPatternId\":\"0ed79425-3edd-4496-9565-6eb60db97434\"}},\"currentIndexPatternId\":\"0ed79425-3edd-4496-9565-6eb60db97434\"},\"indexpattern\":{\"layers\":{}},\"textBased\":{\"layers\":{}}},\"internalReferences\":[],\"adHocDataViews\":{}}},\"enhancements\":{}},\"title\":\"Error code count\"},{\"type\":\"lens\",\"gridData\":{\"x\":0,\"y\":28,\"w\":21,\"h\":14,\"i\":\"4de2bd45-28b0-4e35-a752-aebb69700bc0\"},\"panelIndex\":\"4de2bd45-28b0-4e35-a752-aebb69700bc0\",\"embeddableConfig\":{\"attributes\":{\"title\":\"\",\"visualizationType\":\"lnsHeatmap\",\"type\":\"lens\",\"references\":[{\"type\":\"index-pattern\",\"id\":\"0ed79425-3edd-4496-9565-6eb60db97434\",\"name\":\"indexpattern-datasource-layer-3f68c3e4-801e-474c-8d03-fd5862228453\"}],\"state\":{\"visualization\":{\"shape\":\"heatmap\",\"layerId\":\"3f68c3e4-801e-474c-8d03-fd5862228453\",\"layerType\":\"data\",\"legend\":{\"isVisible\":true,\"position\":\"right\",\"type\":\"heatmap_legend\"},\"gridConfig\":{\"type\":\"heatmap_grid\",\"isCellLabelVisible\":false,\"isYAxisLabelVisible\":true,\"isXAxisLabelVisible\":true,\"isYAxisTitleVisible\":false,\"isXAxisTitleVisible\":false},\"valueAccessor\":\"b0ad50c8-3ebb-4699-abb3-4bac35d29f0f\",\"xAccessor\":\"a0340092-9bbf-4a11-b661-0e1e88f72e6d\",\"yAccessor\":\"ad0cd39d-2afe-42f7-bd75-7f159f1620da\",\"palette\":{\"type\":\"palette\",\"name\":\"positive\",\"params\":{\"steps\":5,\"continuity\":\"above\",\"reverse\":false,\"rangeMin\":0,\"rangeMax\":null,\"name\":\"positive\",\"stops\":[{\"color\":\"#d6e9e4\",\"stop\":0},{\"color\":\"#aed3ca\",\"stop\":20},{\"color\":\"#85bdb1\",\"stop\":40},{\"color\":\"#5aa898\",\"stop\":60},{\"color\":\"#209280\",\"stop\":80}],\"rangeType\":\"percent\"},\"accessor\":\"b0ad50c8-3ebb-4699-abb3-4bac35d29f0f\"}},\"query\":{\"query\":\"\",\"language\":\"kuery\"},\"filters\":[],\"datasourceStates\":{\"formBased\":{\"layers\":{\"3f68c3e4-801e-474c-8d03-fd5862228453\":{\"columns\":{\"a0340092-9bbf-4a11-b661-0e1e88f72e6d\":{\"label\":\"Status Code\",\"dataType\":\"string\",\"operationType\":\"terms\",\"scale\":\"ordinal\",\"sourceField\":\"status_code.keyword\",\"isBucketed\":true,\"params\":{\"size\":5,\"orderBy\":{\"type\":\"alphabetical\",\"fallback\":true},\"orderDirection\":\"asc\",\"otherBucket\":true,\"missingBucket\":false,\"parentFormat\":{\"id\":\"terms\"},\"include\":[],\"exclude\":[],\"includeIsRegex\":false,\"excludeIsRegex\":false},\"customLabel\":true},\"b0ad50c8-3ebb-4699-abb3-4bac35d29f0fX0\":{\"label\":\"Part of Count ff Requests\",\"dataType\":\"number\",\"operationType\":\"count\",\"isBucketed\":false,\"scale\":\"ratio\",\"sourceField\":\"___records___\",\"params\":{\"emptyAsNull\":false},\"customLabel\":true},\"b0ad50c8-3ebb-4699-abb3-4bac35d29f0f\":{\"label\":\"Count of Requests\",\"dataType\":\"number\",\"operationType\":\"formula\",\"isBucketed\":false,\"scale\":\"ratio\",\"params\":{\"formula\":\"count()\",\"isFormulaBroken\":false},\"references\":[\"b0ad50c8-3ebb-4699-abb3-4bac35d29f0fX0\"],\"customLabel\":true},\"ad0cd39d-2afe-42f7-bd75-7f159f1620da\":{\"label\":\" (Hourly Basis)\",\"dataType\":\"date\",\"operationType\":\"date_histogram\",\"sourceField\":\"@timestamp\",\"isBucketed\":true,\"scale\":\"interval\",\"params\":{\"interval\":\"h\",\"includeEmptyRows\":true,\"dropPartials\":false},\"customLabel\":true}},\"columnOrder\":[\"ad0cd39d-2afe-42f7-bd75-7f159f1620da\",\"a0340092-9bbf-4a11-b661-0e1e88f72e6d\",\"b0ad50c8-3ebb-4699-abb3-4bac35d29f0f\",\"b0ad50c8-3ebb-4699-abb3-4bac35d29f0fX0\"],\"incompleteColumns\":{},\"sampling\":1}}},\"indexpattern\":{\"layers\":{}},\"textBased\":{\"layers\":{}}},\"internalReferences\":[],\"adHocDataViews\":{}}},\"enhancements\":{}},\"title\":\"Status Code By Hour of Day\"}]","timeRestore":false,"title":"{{dashboard_name}}","version":1},"coreMigrationVersion":"8.8.0","created_at":"2024-04-19T18:37:14.196Z","id":"a68f407c-92f4-4770-8458-451822fa0d93","managed":false,"references":[{"id":"0ed79425-3edd-4496-9565-6eb60db97434","name":"06b54e54-4396-4a1e-a461-92a016b73cc8:indexpattern-datasource-layer-5e296f9c-d16f-4861-9a5c-16687ce65c47","type":"index-pattern"},{"id":"cd1cbacf-d2e0-4eff-8dc2-97c478198e70","name":"3b1c8f1e-1fd4-4bb0-a342-3ec35ce30249:panel_3b1c8f1e-1fd4-4bb0-a342-3ec35ce30249","type":"visualization"},{"id":"0ed79425-3edd-4496-9565-6eb60db97434","name":"82838041-2c93-4121-b77e-e88441852351:indexpattern-datasource-layer-53c4bb90-1645-4bd5-8051-af52a9503657","type":"index-pattern"},{"id":"0ed79425-3edd-4496-9565-6eb60db97434","name":"6c8f2103-5131-4f19-9d06-2921d98fb7fe:indexpattern-datasource-layer-81e66199-62be-4aec-ac23-4caf81af88c5","type":"index-pattern"},{"id":"0ed79425-3edd-4496-9565-6eb60db97434","name":"4de2bd45-28b0-4e35-a752-aebb69700bc0:indexpattern-datasource-layer-3f68c3e4-801e-474c-8d03-fd5862228453","type":"index-pattern"}],"type":"dashboard","typeMigrationVersion":"8.9.0","updated_at":"2024-04-19T18:37:14.196Z","version":"WzI4NywyXQ=="}
{"excludedObjects":[],"excludedObjectsCount":0,"exportedCount":3,"missingRefCount":0,"missingReferences":[]}
}
```
</details>

***

### defaults file (main.yml)

This YAML configuration sets parameters for the Dashboard and Attendance monitoring. It specifies the Data source UID , Dashboard title , Dashboard UID , and .json file  path .

<details>
<summary> Click here to see main.yml file</summary>
<br>
  
```shell
---
json_file_template_src: "Kibana-attendance-API.ndjson.j2"
json_file_dest: "/home/ubuntu/Kibana-attendance-API.ndjson"
kibana_dashboard_import_url: "http://18.212.132.77:5601/api/saved_objects/_import?overwrite=true"
dashboard_name: "Attendance API Dashboard"
```
</details>

***

# Output

### Role Execution
<img width="1020" alt="Screenshot 2024-04-20 at 4 11 54 AM" src="https://github.com/CodeOps-Hub/Ansible/assets/156056349/2bac265c-eaad-4d12-a0c3-c541bafc8154">

***

### Target Server 

<img width="1186" alt="Screenshot 2024-04-20 at 3 56 40 AM" src="https://github.com/CodeOps-Hub/Ansible/assets/156056349/e90cde9a-6c72-4357-96e2-43a8339687f2">

***

### Kibana Dashboard View

<img width="1319" alt="Screenshot 2024-04-20 at 3 59 12 AM" src="https://github.com/CodeOps-Hub/Ansible/assets/156056349/bed6fb77-b454-40eb-8323-243dad191f17">

***

# Conclusion

This guide outlines the steps to seamlessly integrate the **Attendance Kibana Dashboard** using Ansible. Following these instructions would help you to efficiently visualize and oversee your Attendance API data view of your logs.

***
## Contact Information

|     Name         | Email  |
| -----------------| ------------------------------------ |
| Vidhi Yadav    | vidhi.yadhav.snaatak@mygurukulam.co |
***

## References

| Description                                   | References  |
| --------------------------------------------  | -------------------------------------------------|
|  Ansible Doc | [Reference link](https://docs.ansible.com/ansible/latest/index.html) |
| Reference Link For Ansible Dynamic Inventory | [Reference link](https://devopscube.com/setup-ansible-aws-dynamic-inventory/) |
| Kibana Dashboard| [Reference Link](https://www.youtube.com/watch?v=iNxt0duWBZM&t=253s)|
