+++
title = "Projects & job templates"
weight = 3
+++

A Tower **Project** is a logical collection of Ansible Playbooks. You can manage your playbooks by placing them into a source code management (SCM) system supported by Tower, including Git, Subversion, and Mercurial.

You should definitely keep your Playbooks under version control. In this lab we’ll use Playbooks that are provided in a Github repository.

## Setup Git Repository

For this lab we will use playbooks stored in this Git repository, using the [proper tag/branch](https://github.com/ansible/workshop-examples/tree/summit_2020):

[https://github.com/ansible/workshop-examples](https://github.com/ansible/workshop-examples)

A Playbook to install the Apache webserver has already been committed to the directory **rhel/apache**, `apache_install.yml`, here for reference:

```yaml
---
- name: Apache server installed
  hosts: all

  tasks:
  - name: latest Apache version installed
    yum:
      name: httpd
      state: latest

  - name: latest firewalld version installed
    yum:
      name: firewalld
      state: latest

  - name: firewalld enabled and running
    service:
      name: firewalld
      enabled: true
      state: started

  - name: firewalld permits http service
    firewalld:
      service: http
      permanent: true
      state: enabled
      immediate: yes

  - name: Apache enabled and running
    service:
      name: httpd
      enabled: true
      state: started
```

{{% notice tip %}}
Note the difference to other Playbooks you might have written\! Most importantly there is no `become` and `hosts` is set to `all`.
{{% /notice %}}

To configure and use this repository as a **Source Control Management (SCM)** system in Tower you have to create a **Project** that uses the repository

## Create the Project

- Go to **RESOURCES → Projects** in the side menu view click the ![plus](../../images/green_plus.png?classes=inline) button. Fill in the form:

- **NAME:** Ansible Workshop Examples

- **ORGANIZATION:** Default

- **SCM TYPE:** Git

Now you need the URL to access the repo. You could get the URL in Github as **Clone URL**. Enter the URL into the Project configuration:

- **SCM URL:** `https://github.com/ansible/workshop-examples.git`

- **SCM BRANCH/TAG/COMMIT:** `summit_2020`

- **SCM UPDATE OPTIONS:** Tick all four boxes to always get a fresh copy of the repository and to update the repository when launching a job.

- Click **SAVE**

The new Project will be synced automatically after creation. But you can also do this manually: Sync the Project again with the Git repository by going to the **Projects** view and clicking the circular arrow **Get latest SCM revision** icon to the right of the Project.

After starting the sync job, go to the **Jobs** view: there is a new job for the update of the Git repository.

## Create a Job Template and Run a Job

A job template is a definition and set of parameters for running an Ansible job. Job templates are useful to execute the same job many times. So before running an Ansible **Job** from Tower you must create a **Job Template** that pulls together:

- **Inventory**: On what hosts should the job run?

- **Credentials** What credentials are needed to log into the hosts?

- **Project**: Where is the Playbook?

- **What** Playbook to use?

Okay, let’s just do that: Go to the **Templates** view, click the ![plus](../../images/green_plus.png?classes=inline) button and choose **Job Template**.

{{% notice tip %}}
Remember that you can often click on magnifying glasses to get an overview of options to pick to fill in fields.
{{% /notice %}}

- **NAME:** Install Apache

- **JOB TYPE:** Run

- **INVENTORY:** Workshop Inventory

- **PROJECT:** Ansible Workshop Examples

- **PLAYBOOK:** `rhel/apache/apache_install.yml`

- **CREDENTIAL:** Workshop Credentials

- We need to run the tasks as root so check **Enable privilege escalation**

- Click **SAVE**

You can start the job by directly clicking the blue **LAUNCH** button, or by clicking on the rocket in the Job Templates overview. After launching the Job Template, you are automatically brought to the job overview where you can follow the playbook execution in real time:

![job execution](../../images/job_overview.png)

Since this might take some time, have a closer look at all the details provided:

- All details of the job template like inventory, project, credentials and playbook are shown.

- Additionally, the actual revision of the playbook is recorded here - this makes it easier to analyse job runs later on.

- Also the time of execution with start and end time is recorded, giving you an idea of how long a job execution actually was.

- On the right side, the output of the playbook run is shown. Click on a node underneath a task to get detailed information for the task.

After the Job has finished go to the main **Jobs** view: All jobs are listed here, you should see directly before the Playbook run an SCM update was started. This is the Git update we configured for the **Project** on Job Template launch\!

## Challenge Lab: Check the Result

Time for a little challenge:

- Use an ad hoc command on all hosts to make sure Apache has been installed and is running.

You have already been through all the steps needed, so try this for yourself.

{{% notice tip %}}
What about `systemctl status httpd`?
{{% /notice %}}

<details><summary>**>>Click here for Solution<<**</summary>
<p>

- Go to **Inventories** → **Workshop Inventory**

- In the **HOSTS** view select all hosts and click **RUN COMMANDS**

- **MODULE:** command

- **ARGUMENTS:** systemctl status httpd

- **MACHINE CREDENTIALS:** Workshop Credentials

- Click **LAUNCH**

</p>
</details>

It should all look good!

## What About Some Practice?

Here is a list of tasks:

{{% notice warning %}}
Please make sure to finish these steps as the next chapter depends on it!
{{% /notice %}}

- Create a new inventory called `Webserver` and make only `node1` member of it.

- Copy the `Install Apache` template using the copy icon in the **Templates** view

- Change the name to `Install Apache Ask`

- Change the **INVENTORY** setting of the Project so it will prompt for the inventory on launch (tick the appropriate box).

- **SAVE**

- Launch the `Install Apache Ask` template.

- It will now ask for the inventory to use, choose the `Webserver` inventory and click **NEXT** and **LAUNCH**

- Wait until the Job has finished and make sure it ran only on `node1`

{{% notice tip %}}
The Job didn’t change anything because Apache was already installed in the latest version.
{{% /notice %}}
