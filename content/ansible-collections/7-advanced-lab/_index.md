+++
title = "Advanced Lab"
weight = 7
+++

This is an optional advanced lab for those who feel like they didn't learn enough. In this lab, we will use the previously created Ansible Collection and use it with Ansible Tower. For this we need to run through some additional steps:

1. upload the collection

1. add a requirements file to tell Ansible Tower from where to fetch the Ansible Collection

1. create a playbook which uses the Ansible Collection

Since this is an advanced lab, we will not go through all the steps in details. The following instructions will however still give you some clues.

## Upload the collection

Ansible Tower can download and install Ansible Collections automatically. To use this feature, we have to provide the Ansible Collection by a method which is accessible for Ansible Tower. Since we already have a local web server running, we will simply copy the file into the document root of our existing and already running server.

```bash
[{{< param "control_prompt" >}} ~]$ cd ~/exercise-05/ansible_collections/redhat/workshop_demo_collection/
[{{< param "control_prompt" >}} workshop_demo_collection ]$ sudo cp redhat-workshop_demo_collection-1.0.0.tar.gz /var/lib/awx/public/static/
# verify the download works
[{{< param "control_prompt" >}} workshop_demo_collection ]$ wget -O /dev/null --no-check-certificate http://ansible-1/static/redhat-workshop_demo_collection-1.0.0.tar.gz
--2020-10-09 08:29:23--  http://ansible-1/static/redhat-workshop_demo_collection-1.0.0.tar.gz
Resolving ansible-1 (ansible-1)... 172.16.178.22
Connecting to ansible-1 (ansible-1)|172.16.178.22|:80... connected.
HTTP request sent, awaiting response... 301 Moved Permanently
Location: https://ansible-1:443/static/redhat-workshop_demo_collection-1.0.0.tar.gz [following]
--2020-10-09 08:29:23--  https://ansible-1/static/redhat-workshop_demo_collection-1.0.0.tar.gz
Connecting to ansible-1 (ansible-1)|172.16.178.22|:443... connected.
The certificate's owner does not match hostname ‘ansible-1’
HTTP request sent, awaiting response... 200 OK
Length: 4067 (4.0K) [application/octet-stream]
Saving to: ‘/dev/null’

/dev/null                                                100%[==================================================================================================================================>]   3.97K  --.-KB/s    in 0s

2020-10-09 08:29:23 (310 MB/s) - ‘/dev/null’ saved [4067/4067]
```

## Add a requirements file

You can use your existing `tower_collections` example from the previous part of this lab and adjust it accordingly.

Change the `collections/requirements.yml` to look like the example below:

```yaml
---
collections:
- ansible.posix
- name: redhat.demo_collection
  source: http://{{< param "external_tower1" >}}/static/redhat-workshop_demo_collection-1.0.0.tar.gz
```
