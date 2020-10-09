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

sudo cp redhat-workshop_demo_collection-1.0.0.tar.gz /var/lib/awx/public/static/

## Add a requirements file

You can use your existing `tower_collections` example from the previous part of this lab and adjust it accordingly.

Change the `collections/requirements.yml` to look like the example below:

```yaml
---
collections:
- ansible.posix
- name: redhat.demo_collection
  source: git+ssh://git@ansible-1/projects/demo_collection.git
```