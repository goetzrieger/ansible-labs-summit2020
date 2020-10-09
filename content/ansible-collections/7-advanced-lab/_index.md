+++
title = "Advanced Lab"
weight = 7
+++

This is an optional advanced lab for those who feel like they didn't learn enough. In this lab, we will use the previously created Ansible Collection and use it with Ansible Tower. For this we need to run through some additional steps:

1. create a new project on the git server

1. push the Ansible Collection into Git

1. add a requirements file to tell Ansible Tower from where to fetch the Ansible Collection

1. create a playbook which uses the Ansible Collection

Since this is an advanced lab, we will not go through all the steps in details. The following instructions will however still give you some clues.

## Create a new git project

In a previous chapter of this lab, you downloaded a playbook called `simple_git.yml` to setup a local git server and an initial repository. You can run the same playbook to create a second git repository to push the newly created Ansible Collection.

```bash
[{{< param "control_prompt" >}} ~]$ ansible-playbook simple_git.yml -e git_project=demo_collection
```

## Push your Ansible Collection to git

To be able to use the Ansible Collection from a playbook running on Ansible Tower, we have to initialize and push its content to the git repository.

```bash
[{{< param "control_prompt" >}} workshop_demo_collection]$ git init
[{{< param "control_prompt" >}} workshop_demo_collection]$ git add .
[{{< param "control_prompt" >}} workshop_demo_collection]$ git remote add origin git@ansible-1:projects/demo_collection
[{{< param "control_prompt" >}} workshop_demo_collection]$ git push --set-upstream origin master
```

## Add a requirements file

You can use your existing `tower_collections` example from the previous part of this lab and adjust it accordingly.

Change the `collections/requirements.yml` to look like the example below:

```yaml
---
collections:
- ansible.posix
- name: git+ssh://git@ansible-1/projects/demo_collection
```